# Custom Agent Reporting – Architecture

[![Custom Agent Reporting – Architecture: tenant-wide reporting on AI agents with Power BI, Dataverse, Azure Resource Graph and the Purview audit log](docs/assets/og.png)](https://ryanbowie.github.io/custom-agent-reporting-architecture/)

A community reference architecture for **tenant-wide reporting on AI agents** across Microsoft Copilot Studio, Microsoft 365 Copilot Agent Builder, SharePoint agents and Microsoft Foundry (formerly Azure AI Foundry), built with Power BI, Azure Resource Graph, Dataverse, the Microsoft Purview audit log and Microsoft Defender.

**Read the full guide: <https://ryanbowie.github.io/custom-agent-reporting-architecture/>**

> [!IMPORTANT]
> **This is not a replacement for Microsoft Agent 365.**
>
> - A custom build like this has to be designed, secured, run and maintained by your own organisation. Agent 365 is delivered and maintained by Microsoft as a service.
> - This build only does **reporting**. Agent 365 goes well beyond reporting, into areas such as the agent registry, access control and security. See the [Agent 365 overview](https://learn.microsoft.com/microsoft-agent-365/overview).
> - If you have Agent 365, its registry and APIs are the better inventory source. Start there.
>
> This repository exists to show what is possible with a custom solution, not to recommend one over a Microsoft service.

## What's in this repository

| Path | Contents |
|---|---|
| `docs/index.html` | The guide, published with GitHub Pages: architecture, data lanes, collectors, gateway-free refresh, semantic model, origin and risk scoring, coverage, findings, Shadow AI, permissions and limitations |
| `docs/assets/` | Six screenshots of the working report, and the social card used by the guide |
| `SECURITY.md` | How to report a security problem, and the security model in brief |

**Documentation only.** The Power BI report file, the semantic model and the code that generates them are **not** published. The guide describes the design in enough detail to build your own.

## Screenshots

Six of the 16 report pages from a working build. Names are fictitious and figures are indicative only.

| | |
|---|---|
| ![Executive overview report page with estate totals, platform and activity donuts, and agents created over time](docs/assets/01-executive-overview.png) **Executive overview**: the whole estate on one page, sliced by origin, platform, environment and risk. | ![Agent inventory report page with slicers, creator and model charts, and an agent register table](docs/assets/02-agent-inventory.png) **Agent inventory**: a register with creator, owner, environment, orchestration, model, authentication and risk band. |
| ![Copilot interactions report page with an interaction trend, traffic donut and client surfaces](docs/assets/03-copilot-interactions.png) **Copilot interactions**: per-agent usage from the audit log, including autonomous runs and SharePoint agents. | ![Governance and risk report page with a risk band donut, authentication modes and the highest-risk agents](docs/assets/04-governance-risk.png) **Governance and risk**: sharing, unauthenticated agents, quarantine and an explainable risk score. |
| ![Knowledge sources report page with sources by type, hosts and a type-by-platform matrix](docs/assets/05-knowledge-sources.png) **Knowledge sources**: SharePoint sites, websites, Dataverse tables, Graph connectors and Azure AI Search indexes. | ![Azure AI Foundry report page with resource categories, regions, SKUs and a Foundry estate table](docs/assets/06-azure-ai-foundry.png) **Azure AI Foundry** (now Microsoft Foundry): regions, SKUs and public network exposure. |

## Architecture at a glance

```mermaid
flowchart TB
  subgraph S["1 · Sources"]
    ARG["Azure Resource Graph<br/>PowerPlatformResources · Foundry"]
    AUD["Purview audit log<br/>Microsoft Graph audit queries"]
    ENV["Dataverse in each environment<br/>botcomponents"]
    DEF["Defender advanced hunting"]
  end
  subgraph C["2 · Collectors · Power Automate, daily, in one reporting environment"]
    LC["Lane C · Copilot Interaction Logging<br/>separate build guide"]
    LE["Lane E · SharePoint agents"]
    LD["Lane D · Knowledge sweep"]
  end
  STORE[("Reporting Dataverse · one environment<br/>collector tables · systemuser")]
  subgraph M["3 · Model and report"]
    SM["Power BI semantic model<br/>21 tables · 166 measures"]
    RPT["Power BI report<br/>16 pages · 287 visuals"]
  end
  AUD --> LC
  AUD --> LE
  ENV --> LD
  LC --> STORE
  LE --> STORE
  LD --> STORE
  ARG -- "Lane A · Azure Resource Graph connector" --> SM
  STORE -- "Lanes B to E · one OData feed" --> SM
  DEF -- "Lane F · Web connector" --> SM
  SM --> RPT
```

| Lane | Collects | How |
|---|---|---|
| A | Agent inventory: Copilot Studio and Agent Builder agents, Foundry projects, agent flows, environments | Power BI queries Azure Resource Graph at refresh |
| B | Creator and owner names | Dataverse `systemuser` at refresh |
| C | Per-turn interaction telemetry from the Purview audit log | Daily collector flow (plus a manual back-fill), built with [Copilot Interaction Logging](https://github.com/RyanBowie/copilot-interaction-logging) |
| D | Named knowledge sources for each agent | Daily sweep of every environment, copied into the reporting environment |
| E | SharePoint agent (`.agent` file) inventory | Daily collector flow |
| F | Unsanctioned AI tools on managed devices | Defender advanced hunting at refresh |

> [!NOTE]
> **Copilot interaction usage is captured by [Copilot Interaction Logging](https://ryanbowie.github.io/copilot-interaction-logging/).** Its two flows write audit metadata to three Dataverse tables, and the semantic model reads the interactions table to build Interaction and CopilotSurface. The other two tables monitor the collector. It's one example of a capture method: the same audit records could reach the model from the Microsoft Sentinel `CopilotActivity` table, the Office 365 Management Activity API or a store you already run.

- **Read-only.** It never changes an agent. The only writes are the collectors' rows in your own Dataverse.
- **One reporting environment.** The collectors run in one environment and write there, and lane D copies every environment's knowledge sources into it, so the model needs one Dataverse connection with a static URL, however many environments the tenant has. Install Copilot Interaction Logging in the same environment.
- **No gateway.** Scheduled refresh runs entirely in the Power BI service, using three connectors.
- **Data first.** The report pages were designed around what the APIs actually return, and the guide records what was learned along the way.

## Which table feeds which page

The model has 21 tables and 13 relationships, with Agent at the hub. "Direct" lanes are read by the page's own visuals; "through Agent" lanes arrive through Agent's calculated columns or measures (creator names from lane B, last activity from lane C). Reference tables are static and held in the model. The [full lineage](https://ryanbowie.github.io/custom-agent-reporting-architecture/#lineage) lists every table and where it comes from.

| Page | Visuals | Direct lanes | Through Agent |
|---|---|---|---|
| 1. Executive overview | 23 | A, C | B |
| 2. All agents | 17 | A, C | B |
| 3. Agent analytics | 16 | A, C | B |
| 4. Agent inventory | 17 | A | B, C |
| 5. Creators & ownership | 18 | A | B, C |
| 6. Usage & popularity | 18 | A, Reference | B, C |
| 7. Copilot interactions | 18 | A, C | – |
| 8. SharePoint agents | 19 | C, E | – |
| 9. Adoption & activity | 20 | A | B, C |
| 10. Governance & risk | 20 | A, Reference | B |
| 11. Tools & integration | 18 | A | B |
| 12. Knowledge & grounding | 19 | A | B |
| 13. Knowledge sources | 19 | A, D | B |
| 14. Azure AI Foundry | 19 | A | B |
| 15. Data sources & gaps | 12 | A, Reference | – |
| 16. Shadow AI (endpoints) | 14 | F, Reference | – |

## Related

- [Copilot Interaction Logging](https://github.com/RyanBowie/copilot-interaction-logging) ([guide](https://ryanbowie.github.io/copilot-interaction-logging/)): the lane C collector and the capture method behind the model's Usage tables, documented separately as a build guide with no package to install. It covers every action in both flows and why it exists, the secret-handling decision and the environment variables.
- [Microsoft Agent 365 overview](https://learn.microsoft.com/microsoft-agent-365/overview).

## Licence and disclaimer

Released under the [MIT licence](LICENSE). To report a security problem, see [Security](SECURITY.md).

This is an independent community project. It isn't a Microsoft product and isn't supported by Microsoft. The content is provided "as is", without warranty of any kind. Test any design in a non-production tenant first, and review it with your security and compliance teams.

Microsoft, Microsoft 365, Copilot, Copilot Studio, Power BI, Power Automate, Power Platform, Dataverse, Azure, Microsoft Entra, Microsoft Purview, Microsoft Defender, Microsoft Foundry and SharePoint are trademarks of the Microsoft group of companies.

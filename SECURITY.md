# Security

## Reporting a vulnerability

Please **don't** open a public issue for a security problem. Report it privately through GitHub's [private vulnerability reporting](https://docs.github.com/code-security/security-advisories/guidance-on-reporting-and-writing-information-about-vulnerabilities/privately-reporting-a-security-vulnerability): open the repository's **Security** tab and select **Report a vulnerability**.

This is a personal community project, maintained on a best-effort basis. There is no SLA for responses or fixes.

Never include secrets, tenant IDs, user names, audit records or other personal data in a report or an issue.

## Scope

This repository is an architecture showcase with anonymised screenshots. No Power BI solution is provided: there's no Power BI file, semantic model, flow export or code to patch. Report problems with the guide itself, for example:

- advice that would leave a build insecure or over-permissioned;
- a screenshot or example that exposes something it shouldn't.

You own, secure and maintain whatever you build from the guide.

## Security model in brief

Read [Permissions](https://ryanbowie.github.io/custom-agent-reporting-architecture/#permissions) and [Limitations](https://ryanbowie.github.io/custom-agent-reporting-architecture/#limitations) in the guide before you build. In short:

- **Privileged roles and app registrations.** The report owner needs admin-level read roles, such as Power Platform Administrator. The interaction and SharePoint-agent collectors (lanes C and E) use an app registration with the tenant-wide Microsoft Graph **application** permission `AuditLogsQuery.Read.All`. Get the approvals your organisation requires before anyone grants admin consent, and protect each credential like a privileged admin credential.
- **Lane D is the most privileged.** Its optional provisioning flow adds an application user with **System Administrator** to each environment. Keep the list of grants so that you can revoke them, or grant access per environment by hand instead.
- **Lane C has its own guide.** Interaction telemetry is captured with [Copilot Interaction Logging](https://ryanbowie.github.io/copilot-interaction-logging/), whose [build guide](https://ryanbowie.github.io/copilot-interaction-logging/#build) covers the approvals to get first, the secret-handling decision and run-history exposure.
- **Personal data.** The data holds user principal names, agent creators and owners, and resource names. The lane C tables also hold client IP addresses. Limit who can read the Dataverse tables, the flow run history, the semantic model and the report.
- **Reporting only.** It can't create, edit, turn off, delete or run agents, and it doesn't read prompts or responses. It is not a replacement for Microsoft Agent 365.

## Credential hygiene

- Set an expiry on every client secret or certificate, name an owner and plan rotation.
- Prefer Azure Key Vault or certificates over secrets held in plain-text environment variables.
- Monitor each app's sign-ins, and consider Conditional Access for workload identities.
- If a secret is exposed, delete it from the app registration straight away, create a new one, and review the app's sign-in logs.

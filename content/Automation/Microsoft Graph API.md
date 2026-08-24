# Microsoft Graph API

## Overview

**Microsoft Graph** is the unified REST API surface for Microsoft 365, Entra ID, and much of Azure identity data. Every automation path in this vault — the Entra admin center itself, the Microsoft Graph PowerShell SDK, and any custom script or app you write — ultimately talks to Entra ID **through Graph**, not around it.

> [!important] SC-300 tie-in
> "Automate bulk operations by using the Microsoft Entra admin center and PowerShell" is an explicit Domain 1 exam bullet ([[Entra Identities]]). Graph API is the layer underneath both of those tools — understanding it is what makes the PowerShell cmdlets in [[PowerShell for Entra ID]] make sense rather than feeling like memorized syntax.

Related:

- [[Entra Identities]]
- [[App Registrations]]
- [[PowerShell for Entra ID]]
- [[Privileged Identity Management]]

---

# Graph as the Common Layer

```text
Entra Admin Center (Portal UI)
Microsoft Graph PowerShell SDK
Custom Scripts / Apps (any language)
Third-Party Integrations (e.g., SCIM provisioning)
        │
        ▼ (all of the above call...)
Microsoft Graph API
(https://graph.microsoft.com)
        │
        ▼
Entra ID Directory Data
```

> [!important] SC-300 tie-in
> This "everything goes through Graph" mental model resolves a lot of exam ambiguity — if a scenario asks how to accomplish something programmatically and the answer options include a Graph API call, a PowerShell cmdlet, and a portal action, more than one may be technically correct since the PowerShell SDK is just a Graph client. Pick based on what the scenario emphasizes (automation at scale → PowerShell/Graph; one-off interactive task → portal).

---

# API Versions

|Version|Status|
|---|---|
|`v1.0`|Stable, generally available — production-recommended|
|`beta`|Preview features, including newer capabilities not yet promoted to v1.0|

```text
https://graph.microsoft.com/v1.0/users
https://graph.microsoft.com/beta/users
```

> [!important] SC-300 tie-in
> Some newer governance and workload identity capabilities (certain PIM for Groups operations, newer entitlement management features) may only be exposed in `beta` before reaching `v1.0`. A scenario describing "a feature works in the portal but the documented stable API doesn't support it yet" is pointing at this v1.0/beta gap — not a misconfiguration.

---

# Authentication to Graph

Any app calling Graph — including the PowerShell SDK itself — is an application registered in Entra ID (see [[App Registrations]]), authenticating with either delegated or application permissions.

```text
Calling App (registered in Entra ID)
        │
        ▼
Requests Token for Microsoft Graph
        │
        ├── Delegated Permission ──── Acts as the signed-in user
        └── Application Permission ──── Acts as itself, no user context
        │
        ▼
Graph Validates Token + Permission Grant
        │
        ▼
Request Allowed or Denied
```

> [!important] SC-300 tie-in
> This is the exact same delegated-vs-application distinction covered in [[App Registrations]] — Graph API is where that distinction actually gets *exercised*. A background automation script running unattended (no signed-in user) must use application permissions with admin consent; an interactive PowerShell session run by an admin typically uses delegated permissions scoped to what that admin can already do.

---

# Common Graph Permissions for Identity Administration

|Permission|Purpose|
|---|---|
|`User.ReadWrite.All`|Create/update/delete users|
|`Group.ReadWrite.All`|Manage group objects and membership|
|`RoleManagement.ReadWrite.Directory`|Manage Entra role assignments (including PIM-related operations)|
|`AuditLog.Read.All`|Read sign-in and audit logs|
|`Policy.Read.All` / `Policy.ReadWrite.ConditionalAccess`|Read or manage Conditional Access policies|
|`EntitlementManagement.ReadWrite.All`|Manage catalogs, access packages, and assignments|

> [!important] SC-300 tie-in
> Least privilege applies to Graph permissions the same way it applies everywhere else in this vault — a script that only needs to *read* sign-in logs should be granted `AuditLog.Read.All`, not a broader `Directory.ReadWrite.All`. Exam scenarios testing "what's wrong with this automation's access" sometimes hinge on over-broad permission grants, not a functional bug.

---

# Graph Explorer

A browser-based tool for testing Graph API calls interactively — signing in with a test account, building a request, and inspecting the raw JSON response — without writing a full script first.

```text
Construct Request (GET/POST/PATCH/DELETE + endpoint)
        │
        ▼
Sign In (delegated permissions of the signed-in user)
        │
        ▼
Execute in Graph Explorer
        │
        ▼
Inspect Raw JSON Response
```

Useful for confirming exactly what data a given endpoint returns, and what permissions a call actually requires, before committing to script/SDK code.

---

# Relationship to the PowerShell SDK

The **Microsoft Graph PowerShell SDK** (covered in depth in [[PowerShell for Entra ID]]) is a first-party wrapper around Graph API calls — every cmdlet like `Get-MgUser` or `New-MgGroup` is making the equivalent underlying Graph REST call on your behalf, with PowerShell-friendly parameters and objects.

```text
Get-MgUser -UserId "user@contoso.com"
        │
        ▼ (equivalent to)
GET https://graph.microsoft.com/v1.0/users/user@contoso.com
```

---

# Troubleshooting

## Automation Fails with a Permission/Authorization Error

Check:

1. Whether the calling app/service principal actually has the required Graph permission granted
2. Whether admin consent was granted (required for all Application permissions)
3. Delegated vs. application permission mismatch — confirm which type the scenario actually needs

---

## Feature Available in Portal but Not via Graph v1.0

Check:

1. Whether the operation is only available in the `beta` endpoint
2. Microsoft Graph changelog/documentation for that specific capability's current API status

---

## Script Works Interactively but Fails When Run Unattended

Check:

1. Whether the script was written assuming delegated permissions (requiring an interactive sign-in) but is now running with no user present
2. Whether it needs to be reconfigured to use application permissions with a proper app registration and credential

---

# Best Practices

- Grant the narrowest Graph permission that accomplishes the task — avoid defaulting to broad `.All` write permissions out of convenience
- Use Graph Explorer to validate a call and its required permissions before writing production automation
- Prefer `v1.0` for production scripts; only reach for `beta` when a needed capability genuinely isn't in `v1.0` yet, and treat it as subject to change
- Separate application-permission automation (unattended) from delegated-permission scripts (interactive, run by a specific admin) deliberately, rather than mixing the two patterns in one script

---

# Related Notes

- [[Entra Identities]]
- [[App Registrations]]
- [[PowerShell for Entra ID]]
- [[Privileged Identity Management]]

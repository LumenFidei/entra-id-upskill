# App Registrations

## Overview

An **App Registration** is how an application's identity is defined in Entra ID — its name, supported account types, authentication configuration, and the permissions it can request. This is the "blueprint" side of app identity; the usable, tenant-local instance of that blueprint is the **service principal** (managed administratively as an [[Enterprise Applications|Enterprise Application]]).

> [!important] SC-300 tie-in
> **App Registration vs. Enterprise Application** is the single most important distinction in this domain, and mirrors the Entra roles vs. Azure RBAC confusion from [[Core Terminology]] in terms of how often it's tested. One App Registration (global definition) can have service principals (Enterprise Applications) in multiple tenants if the app is multi-tenant.

Related:

- [[App and Azure Workload Identities]]
- [[Enterprise Applications]]
- [[Authentication Methods]]

---

# App Registration vs. Service Principal / Enterprise Application

```text
App Registration (Global Definition)
        │
        │ (creates a local instance when used in a tenant)
        ▼
Service Principal ── administered as ──► Enterprise Application
   (in Tenant A)                              (in Tenant A)

   Same App Registration, used in another tenant:

Service Principal ── administered as ──► Enterprise Application
   (in Tenant B)                              (in Tenant B)
```

|Aspect|App Registration|Service Principal / Enterprise Application|
|---|---|---|
|Represents|The app's identity definition (blueprint)|The local, permission-holding instance in a specific tenant|
|Where managed|App registrations blade|Enterprise applications blade|
|Holds credentials/secrets|Yes (client secrets, certificates)|No — inherits identity from the registration|
|Holds permission grants/consent|Defines *what can be requested*|Holds *what has actually been granted*|

> [!important] SC-300 tie-in
> A scenario describing "the app needs a new client secret" or "configure the redirect URI" points to the **App Registration**. A scenario describing "assign these specific users to the app" or "review what permissions this app has actually been granted" points to the **Enterprise Application** / service principal side.

---

# Planning for App Registrations

Key decisions before creating a registration:

|Decision|Options|
|---|---|
|Supported account types|Single tenant, multi-tenant, multi-tenant + personal Microsoft accounts|
|Redirect URI type|Web, SPA (single-page app), public client/native|
|Authentication flow|Authorization code, implicit (legacy, discouraged), client credentials (app-only)|

> [!important] SC-300 tie-in
> The **implicit grant flow** is a legacy pattern Microsoft actively discourages in favor of the authorization code flow with PKCE, even for SPAs. If an exam scenario asks "what auth flow should a new single-page application use," the modern, correct answer is authorization code + PKCE, not implicit — treat implicit-flow answer options as distractors unless the scenario explicitly describes a legacy constraint.

---

# Creating App Registrations

```text
Register New Application
        │
        ├── Name
        ├── Supported Account Types
        └── Redirect URI (optional at creation, can add later)
        │
        ▼
App Registration Created
        │
        ▼
Service Principal Automatically Created in Home Tenant
```

---

# Configuring App Authentication

## Redirect URIs

Where Entra ID sends authentication responses (tokens/codes) back to the app. Must be registered exactly — mismatches are a common real-world integration failure.

## Certificates & Secrets

Client secrets (simple shared strings, must be rotated before expiry) or certificates (asymmetric, generally preferred for higher-security scenarios) used by confidential clients (server-side apps) to authenticate themselves when requesting tokens.

```text
Confidential Client (server-side app)
        │
        ▼
Presents Client ID + Secret/Certificate
        │
        ▼
Entra ID Issues Token
```

> [!important] SC-300 tie-in
> Client secrets have an **expiration date** and do not auto-renew — a scenario describing "an application that was working suddenly fails to authenticate with no code changes" is very often pointing at an **expired client secret**, one of the most common real-world (and exam-relevant) app registration failures.

---

# Configuring API Permissions

## Delegated vs. Application Permissions

|Type|Acts As|Requires Signed-In User?|Consent|
|---|---|---|---|
|**Delegated**|The app, acting on behalf of the signed-in user, limited to what that user could do themselves|Yes|User or admin consent|
|**Application**|The app itself, with its own standing permissions, independent of any user|No|Admin consent only|

```text
Delegated Permission

Signed-In User + App Combined Privilege
        │
        ▼
Effective Access = min(User's own access, App's granted permission)

Application Permission

App Acts Alone (no user context)
        │
        ▼
Effective Access = App's granted permission, full stop
```

> [!important] SC-300 tie-in
> This is one of the highest-value distinctions in the entire Workload Identities domain. A scenario describing "a background service needs to read all users' profiles with **no signed-in user present**" requires an **Application permission** — Delegated permissions cannot function without an active user context at all.

## Consent

- **User consent** — an individual user approves permissions for themselves (if allowed by tenant consent settings)
- **Admin consent** — an administrator approves on behalf of the whole organization, required for all Application permissions and often for higher-privilege Delegated permissions

> [!important] SC-300 tie-in
> Application permissions **always require admin consent** — there is no user-consent path for them, since there's no user context to consent on behalf of. If a scenario describes end users being blocked from consenting to an app requesting application-level permissions, that's expected behavior, not a bug to fix.

---

# App Roles

Custom roles defined in an application's manifest (e.g., "Reader," "Approver," "Admin" specific to that app) that can be assigned to users, groups, or **other service principals** (enabling app-to-app authorization, not just human access).

```text
App Manifest Defines App Roles
        │
        ├── Role: "Reader"
        ├── Role: "Approver"
        └── Role: "Admin"
        │
        ▼
Assigned to Users, Groups, or Other Service Principals
        │
        ▼
App Reads Assigned Role(s) from the Token's Claims at Runtime
```

> [!important] SC-300 tie-in
> App roles assigned to **other service principals** (not just users/groups) is the mechanism that enables one application to authorize another application's access to itself at a fine-grained level — this is distinct from simply granting broad API permissions, and is a good exam target for "how do you implement least-privilege app-to-app authorization."

---

# Troubleshooting

## App Suddenly Fails to Authenticate

Check:

1. Client secret/certificate expiration
2. Redirect URI mismatch (especially after an environment/domain change)
3. Whether required admin consent was revoked or never granted for a newly added permission

---

## User Cannot Consent to an App

Check:

1. Whether the app requests Application permissions (no user-consent path exists — needs admin consent)
2. Tenant-wide user consent settings (may be restricted to admin-only consent)
3. Risk-based consent restrictions flagging the app as needing admin review

---

## App-to-App Authorization Not Working as Expected

Check:

1. Whether app roles are actually defined in the resource app's manifest
2. Whether the calling app's service principal has actually been assigned the relevant app role (not just granted a broad API permission)

---

# Best Practices

- Prefer certificates over client secrets where feasible, and track secret expiration proactively rather than reactively
- Use the authorization code flow with PKCE for all new applications; avoid implicit flow
- Apply least privilege deliberately: prefer Delegated over Application permissions whenever a signed-in user context genuinely exists
- Use app roles for app-to-app authorization scenarios instead of over-broad Application permissions

---

# Related Notes

- [[App and Azure Workload Identities]]
- [[Enterprise Applications]]
- [[Authentication Methods]]
- [[Defender for Cloud Apps]]

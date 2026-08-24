# App and Azure Workload Identities

## Overview

**Workload identities** are the non-human identities that applications, scripts, and Azure resources use to authenticate to each other — as distinct from the human user identities covered in [[Entra Identities]]. Picking the *right kind* of workload identity for a given scenario is the core skill this note covers.

> [!important] SC-300 tie-in
> The exam bullet explicitly lists four identity options to choose between: **managed identities, service principals, user accounts, and managed service accounts.** Expect scenario questions where three of the four are plausible-sounding distractors and only one is actually correct for the stated constraints.

Related:

- [[Core Terminology]]
- [[App Registrations]]
- [[Enterprise Applications]]
- [[Identity Protection]]

---

# Choosing the Right Workload Identity

|Identity Type|What It Is|Best Fit|
|---|---|---|
|**Managed Identity**|An identity automatically managed by Entra ID for an Azure resource — no credentials for you to store or rotate|Azure-native workloads (VMs, Function Apps, Logic Apps) accessing other Azure resources|
|**Service Principal**|The local, usable identity of an application (App Registration) within a tenant, with its own credentials (secret or certificate) you manage|Apps needing to authenticate that aren't Azure-hosted, or need cross-tenant/multi-tenant identity|
|**User Account**|A regular human (or "service-account-style" human) identity with a password|Legacy scenarios only — using a human account to run a service is a known anti-pattern|
|**Managed Service Account (on-premises)**|An AD DS **Group Managed Service Account (gMSA)** — a domain-managed account for on-prem Windows services|On-premises Windows services needing domain authentication without manual password management|

```text
Decision Flow (Simplified)

Is the workload running natively in Azure?
        │
       Yes ──► Managed Identity (preferred — no credential management)
        │
        No
        │
        ▼
Does it need its own tenant identity (e.g. multi-tenant SaaS, non-Azure hosting)?
        │
       Yes ──► Service Principal (via App Registration)
        │
        No
        │
        ▼
Is it an on-premises Windows service under AD DS?
        │
       Yes ──► Group Managed Service Account (gMSA)
```

> [!important] SC-300 tie-in
> **Using a regular user account to run a service or script is a documented anti-pattern** the exam expects you to identify and correct — even though it "works," the correct recommendation in almost every scenario is to migrate to a managed identity or service principal instead, since user accounts carry passwords that expire, get MFA-challenged, and are tied to an actual person's lifecycle.

---

# Managed Identities

## System-Assigned vs. User-Assigned

|Type|Lifecycle|Reusability|
|---|---|---|
|System-assigned|Tied 1:1 to the Azure resource — created and deleted with it|Cannot be shared across resources|
|User-assigned|Standalone Azure resource of its own|Can be assigned to multiple Azure resources simultaneously|

```text
System-Assigned Managed Identity

Azure VM Created ──► Identity Created
Azure VM Deleted ──► Identity Deleted
(1:1, no manual identity lifecycle management)

User-Assigned Managed Identity

Identity Created Independently
        │
        ├── Assigned to VM A
        ├── Assigned to Function App B
        └── Assigned to Logic App C
(shared, survives independently of any one resource)
```

> [!important] SC-300 tie-in
> If a scenario describes multiple Azure resources needing to **share the exact same identity and permission set** (e.g., for consistent access across a scaled-out set of Function Apps), that's user-assigned. If the scenario emphasizes simplicity and a strict one-to-one relationship with no reuse, that's system-assigned.

## Creating and Assigning Managed Identities

```text
1. Enable Managed Identity on the Azure Resource
   (system-assigned: toggle on the resource itself)
   (user-assigned: create the identity resource first, then attach it)
        │
        ▼
2. Entra ID Creates a Corresponding Service Principal Automatically
        │
        ▼
3. Grant That Service Principal RBAC/Permissions on Target Resources
        │
        ▼
4. Application Code Requests a Token Using the Managed Identity
   (no credentials stored in code or config)
```

## Using a Managed Identity to Access Other Azure Resources

The resource (e.g., a VM) requests a token from the Azure Instance Metadata Service using its managed identity, then presents that token to the target resource (e.g., Key Vault, Storage). No secret is ever stored in application code or configuration.

```text
Azure VM (Managed Identity)
        │
        ▼
Requests Token from Azure AD (via Instance Metadata Service)
        │
        ▼
Presents Token to Target Resource (e.g., Key Vault)
        │
        ▼
Target Resource Validates Token + Checks RBAC Assignment
        │
        ▼
Access Granted (no credential ever exposed in code)
```

> [!important] SC-300 tie-in
> The core value proposition tested here: managed identities **eliminate credential management entirely** for Azure-to-Azure authentication. A scenario asking "how to let a VM access a Key Vault without storing a secret anywhere" is asking for a managed identity — this is one of the most direct, low-ambiguity exam questions in this domain.

---

# Service Principals (Deeper Detail)

A service principal is created automatically whenever an App Registration is used within a specific tenant (see [[App Registrations]] for the full app-identity lifecycle) — it's the object that actually holds permissions and can authenticate, as opposed to the App Registration, which is more like the app's global blueprint.

> [!important] SC-300 tie-in
> Don't confuse **service principal** (a workload identity concept, this note) with **Enterprise Application** (the administrative surface for managing that same service principal's settings — see [[Enterprise Applications]]). They refer to the same underlying object, viewed through two different administrative lenses.

---

# Troubleshooting

## Application Fails to Authenticate to an Azure Resource

Check:

1. Whether a managed identity is actually enabled on the source resource
2. Whether the managed identity's service principal has the correct RBAC role assignment on the target resource
3. Whether the code is requesting a token correctly (system-assigned vs. user-assigned client ID mismatches are a common code-level bug)

---

## Shared Identity Needed Across Multiple Resources but Currently System-Assigned

This isn't fixable in place — system-assigned identities cannot be converted to user-assigned. The resolution is to create a user-assigned managed identity and reassign permissions to it, then attach it to all the resources that need to share it.

---

## Legacy Script Still Running Under a Human User Account

Check:

1. Whether that account is subject to Conditional Access/MFA policies that could unexpectedly break the automation when enforcement changes
2. Migration path to a managed identity (if Azure-native) or service principal (if not)

---

# Best Practices

- Default to managed identities for any Azure-native workload; avoid service principals with manually managed secrets when a managed identity will do
- Prefer user-assigned managed identities when multiple resources need to share identical permissions, to avoid permission drift across many system-assigned identities
- Never use a human user account to run an automated service or script — treat any discovery of this pattern as a finding requiring remediation
- Rotate service principal credentials (secrets/certificates) on a defined schedule if a managed identity genuinely isn't an option

---

# Related Notes

- [[Core Terminology]]
- [[App Registrations]]
- [[Enterprise Applications]]
- [[Identity Protection]]

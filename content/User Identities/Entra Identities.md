# Entra Identities

## Overview

This note covers the core, day-to-day identity objects an administrator creates and manages: users, groups, custom security attributes, bulk operations, device join/registration, and licensing.

> [!important] SC-300 tie-in
> This is the highest-volume, most operationally "hands-on-keyboard" section of the User Identities domain — expect scenario questions that require picking the right tool (portal vs. PowerShell vs. Graph) for a given task, not just knowing the concept exists.

Related:

- [[Tenant Configuration]]
- [[Core Terminology]]
- [[External Identities]]
- [[Hybrid Identity]]

---

# Users

## Creation Methods

```text
User Creation Paths

Entra Admin Center (one at a time, interactive)
Microsoft Graph PowerShell SDK (New-MgUser, scriptable)
Microsoft Graph API (direct REST calls, automation-friendly)
Bulk CSV Import (Entra Admin Center → Bulk create)
Entra Connect Sync / Cloud Sync (from on-prem AD DS)
```

## Key User Attributes

- User Principal Name (UPN) — the sign-in identity
- Display name, given name, surname
- Usage location — required before license assignment (determines service availability by country/region)
- Manager (used for approval workflows, org chart features)

> [!important] SC-300 tie-in
> **Usage location** is a classic exam trap: a newly created user can exist without it, but license assignment will silently fail or be blocked until usage location is set. If a scenario describes "a new user isn't getting their expected license," check usage location before anything else.

---

# Groups

## Group Types

|Type|Purpose|
|---|---|
|Security Group|Access control — resource permissions, app assignment, licensing|
|Microsoft 365 Group|Collaboration — shared mailbox, calendar, SharePoint site, Teams|

## Membership Types

|Type|Description|
|---|---|
|Assigned|Members are manually added/removed|
|Dynamic User|Membership rule evaluates user attributes automatically|
|Dynamic Device|Membership rule evaluates device attributes automatically|

```text
Dynamic Group Rule Example

(user.department -eq "Sales") -and (user.country -eq "US")
        │
        ▼
Membership Auto-Updates as Attributes Change
```

> [!important] SC-300 tie-in
> Dynamic group membership requires at least a P1 license. A scenario describing a Free-tier tenant that "needs group membership to update automatically as users change departments" has licensing as the actual blocker, not a rule-syntax problem.

---

# Custom Security Attributes

Custom, tenant-defined key-value attributes attached to directory objects (primarily users, and applications for workload identity scenarios), organized into **attribute sets**.

```text
Attribute Set: "Project"
        │
        ├── Attribute: ProjectCode (string)
        ├── Attribute: ClearanceLevel (string, predefined values)
        └── Attribute: ContractEndDate (date)
```

## Use Cases

- Fine-grained, attribute-based scoping in Conditional Access or entitlement management (beyond what standard directory attributes or group membership provide)
- Tagging workload identities (applications/service principals) for governance purposes

> [!important] SC-300 tie-in
> Assigning and managing custom security attributes requires a **separate, dedicated role** (Attribute Assignment Administrator / Attribute Definition Administrator) — even a Global Administrator cannot manage these by default without being assigned (or self-elevating into) one of these specific roles. This separation-of-duties design is intentional and a good exam distractor target.

---

# Bulk Operations

## Entra Admin Center Bulk Actions

- Bulk create users (CSV template)
- Bulk invite guest users (CSV template)
- Bulk delete users

## PowerShell / Microsoft Graph SDK

Scripted equivalents for anything achievable in the portal, plus operations not exposed in bulk UI flows (e.g., complex conditional bulk updates).

```text
Common Cmdlets (Microsoft Graph PowerShell SDK)

New-MgUser
Update-MgUser
Remove-MgUser
New-MgGroup
New-MgGroupMember
```

> [!important] SC-300 tie-in
> The legacy AzureAD and MSOnline PowerShell modules are deprecated in favor of the **Microsoft Graph PowerShell SDK** — if an exam question presents old-module cmdlet syntax as an answer option, treat that as a strong signal it's the distractor, not the correct choice.

---

# Device Join and Registration Management

Administrators manage device join/registration settings at the tenant level (see [[Tenant Configuration]] for the settings themselves) and can view/manage individual device objects — enabling/disabling devices, and reviewing join type per device (registered, joined, hybrid joined — see [[Core Terminology]]).

---

# Licensing: Assign, Modify, Report

## Assignment Models

- **Direct assignment** — per user
- **Group-based licensing** — inherited via group membership (see [[Entra ID Editions and Licensing]])

## Modifying Licenses

Adding/removing individual service plans within a product license (e.g., disabling Yammer or Stream within an M365 SKU for specific users) without removing the whole license.

## License Reporting

Entra admin center provides reports on:

- Users with a given license assigned
- License assignment errors (conflicting service plans, insufficient licenses in a group-based assignment)

> [!important] SC-300 tie-in
> "Assign, modify, and report on licenses" is called out as its own exam bullet — expect at least one question purely about diagnosing a license assignment **error** (not just performing an assignment), most often rooted in group-based licensing conflicts.

---

# Troubleshooting

## New User Not Receiving Expected License

Check:

1. Usage location is set on the user object
2. Group-based licensing assignment errors (conflicting service plans)
3. Available license count in the tenant (not exhausted)

---

## Dynamic Group Not Updating as Expected

Check:

1. Tenant license level supports dynamic groups (P1+)
2. Rule syntax correctness
3. Processing delay — dynamic membership updates are near-real-time but not instantaneous at scale

---

## Bulk Import Partially Fails

Check:

1. CSV template formatting against the current expected schema
2. Duplicate UPNs within the batch or against existing users
3. Row-level error report generated by the bulk operation

---

# Best Practices

- Set usage location as a required field in any onboarding automation, before license assignment is attempted
- Prefer dynamic groups for anything driven by an HR/attribute source of truth; reserve assigned groups for exceptions
- Use custom security attributes sparingly and document the attribute set's purpose — they're powerful but easy to sprawl
- Standardize on the Microsoft Graph PowerShell SDK for scripting; avoid deprecated AzureAD/MSOnline modules in new automation

---

# Related Notes

- [[Tenant Configuration]]
- [[Core Terminology]]
- [[External Identities]]
- [[Hybrid Identity]]
- [[Entra ID Editions and Licensing]]

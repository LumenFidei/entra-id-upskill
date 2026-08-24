# Privileged Identity Management (PIM)

## Overview

**Privileged Identity Management (PIM)** provides just-in-time, time-bound activation of privileged roles — rather than granting standing (always-on) access to sensitive Entra roles, Azure resource roles, or privileged groups.

> [!important] SC-300 tie-in
> PIM requires a **P2** license. This is arguably the single most exam-dense feature in the entire Identity Governance domain — it has its own dedicated exam bullet group and connects backward into [[Tenant Configuration]] (Entra roles) and forward into [[Access Reviews]] (recertifying assignments).

Related:

- [[Tenant Configuration]]
- [[Access Reviews]]
- [[Entra ID Editions and Licensing]]
- [[Core Terminology]]

---

# Eligible vs. Active Assignments

The foundational PIM concept.

|State|Meaning|
|---|---|
|**Eligible**|The user *can* activate the role when needed, but does not have standing permissions right now|
|**Active**|The role's permissions are currently in effect|

```text
Eligible Assignment
        │
        ▼
User Requests Activation (may require MFA, justification, approval)
        │
        ▼
Active Assignment (time-bound, e.g., 2 hours)
        │
        ▼
Automatically Expires Back to Eligible
```

> [!important] SC-300 tie-in
> This directly connects to a callout in [[Tenant Configuration]]: effective permissions at any given moment reflect only **active** assignments, not eligible ones. A scenario describing "why can't this admin perform an action right now, even though they're supposed to have the role" is almost always testing whether you check eligible-vs-active state first.

---

# PIM for Microsoft Entra Roles

Settings configurable per role:

|Setting|Purpose|
|---|---|
|Maximum activation duration|How long an activation remains active before automatically expiring|
|Require MFA on activation|Forces a fresh MFA challenge specifically at activation time|
|Require justification|User must provide a text reason for activating|
|Require approval|Activation requires a designated approver to sign off before taking effect|
|Notification settings|Who gets notified when a role is activated|

```text
Global Administrator Role Settings (Example)
        │
        ├── Max Activation Duration: 2 hours
        ├── Require MFA: Yes
        ├── Require Justification: Yes
        └── Require Approval: Yes (approver: Security Team)
```

> [!important] SC-300 tie-in
> Configuring **max activation duration, MFA on activation, and required justification for Global Administrator** is a frequently cited real-world and exam baseline recommendation — expect a scenario question built almost verbatim around this exact configuration.

---

# PIM for Azure Resources

The same eligible/active model extended to Azure RBAC role assignments (e.g., Contributor on a subscription) — distinct from Entra roles, per the [[Core Terminology]] Entra roles vs. Azure RBAC distinction.

```text
PIM for Azure Resources
        │
        ▼
Eligible Assignment: Contributor on Subscription X
        │
        ▼
User Activates When Needed (time-bound, auditable)
        │
        ▼
Active Assignment Expires Automatically
```

> [!important] SC-300 tie-in
> PIM for Azure resources and PIM for Entra roles are **configured and managed separately** — they share the same eligible/active concept but live in different parts of the admin experience, matching the underlying Entra roles vs. Azure RBAC split. Don't assume configuring one automatically covers the other.

---

# PIM for Groups

Extends just-in-time activation to **membership or ownership of a specific group** — useful when a group itself grants meaningful access (e.g., app role assignment, or a group nested into a privileged role assignment) and standing membership is undesirable.

```text
PIM-for-Groups-Enabled Security Group
        │
        ├── Eligible Member: User A
        └── Eligible Owner: User B
        │
        ▼
User A Activates Membership When Needed
        │
        ▼
Temporarily a Member → Inherits Whatever That Group Grants
```

> [!important] SC-300 tie-in
> PIM for Groups is the answer whenever a scenario describes needing just-in-time access to something that **isn't directly an Entra or Azure role** — for example, a group that grants an app role or SharePoint site access. Applying PIM to the group itself extends just-in-time governance to resources PIM wouldn't otherwise reach directly.

---

# The PIM Request and Approval Process

```text
User Requests Role Activation
        │
        ▼
System Checks Role Settings
(MFA required? Justification required? Approval required?)
        │
        ▼
If Approval Required:
Request Routed to Designated Approver(s)
        │
        ▼
Approver Approves or Denies (with optional comments)
        │
        ▼
If Approved: Role Becomes Active for the Configured Duration
```

Approvers receive notifications and can review pending requests in a dedicated queue; multiple approvers can be configured, and approval can require just one or all of them, depending on configuration.

---

# Analyzing PIM Audit History and Reports

PIM maintains a detailed audit trail of every activation, approval/denial, and assignment change — who requested what, when, with what justification, and who approved it.

> [!important] SC-300 tie-in
> A scenario describing "prove that no unauthorized admin activated Global Administrator during an incident window" is a PIM audit history question — this is distinct from the general Entra audit logs covered in [[Monitoring and Logs]], though both contribute to a full investigation; PIM's audit history specifically has the eligible/active/approval context general audit logs don't capture as cleanly.

---

# Break-Glass Accounts

Emergency access accounts, excluded from Conditional Access policies and **not** managed through PIM's normal eligible/active workflow — they must remain immediately usable even if PIM, Conditional Access, or the organization's normal identity provider is unavailable or misconfigured.

```text
Break-Glass Account
        │
        ├── Excluded from all Conditional Access policies
        ├── Cloud-only (not synced/hybrid, to avoid on-prem dependency)
        ├── Strong, long, monitored credential (not MFA-locked-in a way that could itself fail)
        └── Usage actively monitored/alerted — any sign-in should trigger investigation
```

> [!important] SC-300 tie-in
> Break-glass accounts are deliberately **not** put through standard PIM eligible-activation workflows — the entire point is that they must work even if PIM itself, Conditional Access, or another dependency has failed or locked everyone else out. A scenario suggesting "require PIM activation for the break-glass account too" is describing an anti-pattern, not a security improvement.

---

# Troubleshooting

## Admin Cannot Activate an Eligible Role

Check:

1. Whether MFA has been completed if required for activation
2. Whether approval is required and is still pending
3. Whether the role's activation window/schedule restricts when activation is even possible

---

## Role Activated But Permissions Not Yet Reflected

Check:

1. Token/session refresh timing — a user may need to sign out and back in, or wait for token refresh, for newly active permissions to take effect in some clients
2. Whether the activation actually succeeded (check PIM audit history for confirmation)

---

## Break-Glass Account Inaccessible During an Incident

Check:

1. Whether it was accidentally included in a Conditional Access policy exclusion list update
2. Credential storage/retrieval process — this should be tested periodically, not assumed to work

---

# Best Practices

- Require MFA, justification, and (for the most sensitive roles) approval on activation for all standing privileged Entra roles
- Apply PIM for Groups to any group that indirectly grants meaningful privileged access, not just direct Entra/Azure role assignments
- Pair every privileged role assignment with a recurring access review (see [[Access Reviews]]) to catch stale eligible assignments
- Maintain at least two break-glass accounts, excluded from Conditional Access, with monitored sign-in alerting, and test their usability periodically

---

# Related Notes

- [[Tenant Configuration]]
- [[Access Reviews]]
- [[Entra ID Editions and Licensing]]
- [[Core Terminology]]
- [[Monitoring and Logs]]

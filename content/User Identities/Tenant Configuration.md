# Tenant Configuration

## Overview

Tenant configuration covers the administrative scaffolding of an Entra ID tenant: roles, administrative units, domains, branding, and directory-wide settings. This is foundational — almost every other note in this vault assumes a correctly configured tenant underneath it.

> [!important] SC-300 tie-in
> This maps directly to the first bullet group under "Implement and manage user identities" (20–25% of the exam). Expect direct questions on role scope and administrative unit design, not just "what is a role."

Related:

- [[Core Terminology]]
- [[Entra Identities]]
- [[Privileged Identity Management]]

---

# Built-in vs. Custom Entra Roles

## Built-in Roles

Predefined roles covering common administrative scenarios.

```text
Common Built-in Roles

Global Administrator ──── Full tenant control
User Administrator ──── Manage users, groups, licenses (not roles)
Helpdesk Administrator ──── Reset passwords for non-admins
Conditional Access Administrator ──── Manage CA policies
Application Administrator ──── Manage app registrations & enterprise apps
Privileged Role Administrator ──── Manage role assignments, including PIM
```

## Custom Roles

Built from a defined set of permissions when no built-in role fits precisely. Custom roles support granular permission selection but have a smaller surface area than Azure RBAC custom roles.

> [!important] SC-300 tie-in
> Custom Entra roles can only be created with at least a P1 license, and certain highly sensitive permissions (e.g., anything touching other role assignments) are restricted from custom roles entirely — the exam may test whether a scenario actually requires a built-in role like Privileged Role Administrator instead.

---

# Administrative Units

An **Administrative Unit (AU)** scopes a role assignment to a defined subset of the directory — commonly a department, region, or school (in education tenants).

```text
Directory
   │
   ├── Administrative Unit: "Sales"
   │        └── User Administrator role scoped to only Sales users
   │
   └── Administrative Unit: "Engineering"
            └── Helpdesk Administrator role scoped to only Engineering users
```

## When to Use Administrative Units

- Delegating helpdesk/user administration to regional or departmental IT teams
- Preventing an admin from affecting users outside their assigned scope
- Multi-national orgs needing geographically scoped delegation

## Restricted Management Administrative Units

A special AU type where members (users, groups, or devices) are protected from modification even by administrators with directory-wide roles — only administrators explicitly assigned to the restricted AU can manage them.

> [!important] SC-300 tie-in
> Restricted management AUs are the answer whenever a scenario says "even Global Administrators should not be able to modify this specific set of high-value accounts." Don't reach for Conditional Access or PIM for that requirement — it's an AU feature.

---

# Evaluating Effective Permissions

Entra ID provides a **"Check access"** / effective permissions view (per role, per user) to determine what a given principal can actually do once role assignments, AU scoping, and PIM eligibility are all combined.

```text
Effective Permission =
     Role Definition
        +
     Assignment Scope (Directory-wide or AU)
        +
     Active vs. Eligible (PIM) State
```

> [!important] SC-300 tie-in
> A user can be *eligible* for a role via PIM without it being *active* — effective permissions at any given moment reflect only active assignments. Exam scenarios testing "why can't this admin perform X right now" often hinge on this eligible-vs-active distinction rather than a missing role assignment.

---

# Domains

## Custom Domain Verification

Adding a custom domain (e.g., `company.com`) requires verification via a DNS TXT or MX record before it can be used for usernames or as a primary domain.

```text
Add Custom Domain
        │
        ▼
Add DNS TXT/MX Verification Record
        │
        ▼
Verify in Entra Admin Center
        │
        ▼
Set as Primary Domain (optional)
```

## Primary vs. Additional Domains

The **primary domain** is the default suffix for new user creation; additional verified domains can also be used per-user.

---

# Company Branding

Customizable sign-in experience — logo, background image, and text — shown to users during authentication. Can be configured per-language and, in more advanced setups, targeted by browser language or by whether the sign-in is from a known organizational context.

> [!important] SC-300 tie-in
> Branding is cosmetic and has no security function — don't confuse it with security defaults or Conditional Access controls on a scenario question about "how to warn users about phishing during sign-in." That's closer to a Conditional Access custom message or terms of use, not branding.

---

# Tenant Properties, User, Group, and Device Settings

## Tenant Properties

Tenant name, tenant ID, technical contact, and notification settings.

## User Settings

Examples:

- Whether users can register applications (app registration self-service)
- Whether users can create security groups
- Default user role permissions (e.g., can read other users' directory info)
- Restrict access to the Entra admin center to admins only

## Group Settings

Examples:

- Whether users can create Microsoft 365 groups
- Group naming policies (prefix/suffix enforcement, blocked words)
- Expiration policies for unused groups

## Device Settings

Examples:

- Maximum number of devices per user
- Whether users can register their devices, and whether MFA is required at registration time
- Enrollment restrictions (which platforms are permitted)

> [!important] SC-300 tie-in
> "Users can register applications" defaulting to **On** in many tenants is a commonly tested misconfiguration — a scenario describing unauthorized or shadow app registrations often traces back to this setting rather than anything in Conditional Access or Enterprise Applications.

---

# Troubleshooting

## Admin Cannot Perform an Expected Action

Check:

1. Whether the role assignment is Active or only Eligible (PIM)
2. Whether the target object falls outside the admin's Administrative Unit scope
3. Whether the target object sits in a Restricted Management AU

---

## New Custom Domain Not Usable for Usernames

Check:

1. DNS verification record was added correctly (TXT or MX)
2. Verification was completed in the Entra admin center (not just DNS-side)
3. Domain isn't already claimed by another tenant

---

# Best Practices

- Use Administrative Units for any real delegation need rather than granting directory-wide roles out of convenience
- Reserve custom roles for genuine gaps — built-in roles cover the vast majority of real scenarios and are easier to reason about during audits
- Review "users can register applications" and "users can create groups" settings early — these default settings are a common unmanaged sprawl source
- Use restricted management AUs for break-glass and other highest-value accounts

---

# Related Notes

- [[Core Terminology]]
- [[Entra Identities]]
- [[Privileged Identity Management]]
- [[Monitoring and Logs]]

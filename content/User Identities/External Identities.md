# External Identities

## Overview

**External Identities** covers how Entra ID manages access for users outside the organization — partners, vendors, contractors, and other tenants — without creating full internal accounts for them.

> [!note] Cross-vault reference
> Your **Zscaler** vault has a note, `PRA Entra ID Guest Access.md`, that walks through a real guest-authentication failure caused by exactly the UPN mismatch described below. It's a separate Obsidian vault (and separate Quartz site), so it can't be linked here directly — worth copying or adapting into this vault if you want it cross-referenced natively, since the failure pattern is core SC-300 material, not just a Zscaler-specific gotcha.

> [!important] SC-300 tie-in
> Cross-tenant access settings and B2B collaboration are a frequent source of exam scenario questions involving mergers, acquisitions, or multi-subsidiary organizations — know the difference between B2B collaboration (guest identity in your tenant) and cross-tenant synchronization (automated provisioning across tenants you own).

Related:

- [[Tenant Configuration]]
- [[Entra Identities]]
- [[Identity Protection]]

---

# External Collaboration Settings

Tenant-wide controls governing how guest access behaves by default.

|Setting|Purpose|
|---|---|
|Guest invite restrictions|Who can invite guests (everyone, only specific roles, or nobody)|
|Guest user access restrictions|What guests can see in the directory (same access as members, limited, or most restrictive)|
|Collaboration restrictions|Allowlist or denylist specific external domains for B2B invites|

```text
Guest Invite Restriction Levels

Anyone in the org can invite
        │
Members can invite (guests cannot)
        │
Only users with invite role can invite
        │
No one can invite (invites disabled)
```

> [!important] SC-300 tie-in
> "Guest user access restrictions" defaulting to a more limited view of the directory (compared to members) is a common root cause when a scenario describes a guest unable to see expected colleagues or groups — this is a directory visibility setting, not a Conditional Access or licensing issue.

---

# Inviting External Users

## Individual Invitations

Sent via the Entra admin center, generating an invitation email the guest accepts to redeem access.

## Bulk Invitations

CSV-based bulk invite, mirroring the bulk user creation flow in [[Entra Identities]].

```text
Invite Guest
      │
      ▼
Invitation Email Sent
      │
      ▼
Guest Redeems Invitation
      │
      ▼
Guest Object Created in Tenant (UserType = Guest)
```

---

# Managing External User Accounts

Guest objects behave mostly like member objects but are flagged `UserType: Guest` and typically carry a UPN in the form:

```text
<username>_<homedomain>#EXT#@<resourcetenant>.onmicrosoft.com
```

> [!important] SC-300 tie-in
> This UPN format is the root cause of a well-documented authentication failure pattern when guests need access to any SAML-based clientless app relying on a UPN match (Browser Access-style portals are one real-world example). The fix involves correcting both the Entra provisioning attribute mapping and the SAML Attributes & Claims configuration so the "real" UPN is consistently passed through — see the cross-vault reference above for a full worked example.

---

# Cross-Tenant Access Settings

Governs trust between your tenant and **specific external Entra tenants** — distinct from general B2B collaboration settings, which apply to any external domain.

## Inbound Settings

Controls what users from a specific external tenant can access when they authenticate into your tenant as guests.

## Outbound Settings

Controls what your users can access when authenticating into external partner tenants.

## Organizational vs. Default Settings

|Scope|Applies To|
|---|---|
|Default settings|All external tenants not explicitly configured|
|Organizational settings|A specific external tenant, overriding the default|

```text
Cross-Tenant Access Settings
        │
        ├── Default Settings (baseline for all orgs)
        └── Organization-Specific Settings (override for Org X, Org Y, ...)
```

> [!important] SC-300 tie-in
> Cross-tenant access settings also control **MFA and device claims trust** between tenants — a partner org's MFA claim can be trusted so the guest isn't re-prompted for MFA in your tenant. Exam scenarios describing "guest keeps getting re-prompted for MFA despite already completing it in their home tenant" point to this trust setting, not a Conditional Access misconfiguration in your own tenant.

---

# Cross-Tenant Synchronization

Automates user provisioning **between tenants your organization owns** (e.g., a parent company and subsidiary, or pre/post-merger tenants) — distinct from B2B collaboration, which is invite-based and typically used for external partners.

```text
Tenant A (Source)
      │
      ▼
Cross-Tenant Sync Configuration
      │
      ▼
Tenant B (Target) — Users Auto-Provisioned as Guests (or Members)
```

> [!important] SC-300 tie-in
> Cross-tenant synchronization is a **provisioning automation** feature (push-based, source-of-truth driven) — it is not the same mechanism as manually inviting individual guests, and it's specifically designed for multi-tenant organizations under common ownership, not arbitrary partner collaboration.

---

# External Identity Providers

Beyond standard Entra-to-Entra B2B collaboration, Entra ID supports federating with external identity providers directly:

|Protocol|Use Case|
|---|---|
|SAML|Federating with a partner's non-Entra IdP for guest authentication|
|WS-Fed|Federating with legacy identity systems (e.g., AD FS-based partners)|

```text
Guest User
     │
     ▼
Your Tenant Recognizes External IdP (SAML/WS-Fed)
     │
     ▼
Guest Authenticates Against Their Own IdP
     │
     ▼
Guest Granted Access to Your Resources (as a Guest Object)
```

---

# Troubleshooting

## Guest Cannot Redeem Invitation

Check:

1. Guest invite restrictions — confirm the inviting user has permission
2. Collaboration restrictions — confirm the guest's domain isn't denylisted
3. Whether the invitation email landed in spam/was blocked by the guest's own org

---

## Guest Authentication Fails to a Clientless App (401)

Check:

1. UPN mismatch between what the relying app expects and what Entra ID's SAML claims actually send (see the cross-vault reference note above)
2. Provisioning attribute mapping (`username` → `originalUserPrincipalName`)
3. SAML Attributes & Claims (`Unique User Identifier` → `user.localuserprincipalname`)

---

## Guest Repeatedly Re-Prompted for MFA

Check:

1. Cross-tenant access inbound trust settings for MFA claims from the guest's home tenant
2. Whether the home tenant's MFA claim is even being sent (depends on their own configuration)

---

# Best Practices

- Set collaboration restrictions to an explicit allowlist for regulated environments rather than leaving invites fully open
- Document which partner tenants have cross-tenant access organizational overrides, since defaults can silently differ from what a specific partnership actually needs
- Use cross-tenant synchronization only for tenants under common organizational ownership — not as a shortcut for general partner collaboration
- When onboarding guest access to clientless/SAML apps, validate the UPN mapping early rather than after users start reporting failures

---

# Related Notes

- [[Tenant Configuration]]
- [[Entra Identities]]
- [[Identity Protection]]
- [[Hybrid Identity]]

# Microsoft Entra ID Overview

## Overview

**Microsoft Entra ID** (formerly Azure Active Directory / Azure AD) is Microsoft's cloud-based identity and access management (IAM) service. It authenticates users and authorizes access to Microsoft 365, Azure resources, and thousands of SaaS and custom applications.

Entra ID is the identity plane underneath nearly everything else on the SC-300 exam — Conditional Access, governance, and workload identities all sit on top of it.

> [!important] SC-300 tie-in
> The exam's audience profile explicitly frames the identity and access administrator role around Zero Trust principles — "verify explicitly" for every access decision. Expect this framing to color scenario questions even when the literal question is about a specific feature.

Related:

- [[Entra ID Editions and Licensing]]
- [[Core Terminology]]
- [[Tenant Configuration]]

---

# From Azure AD to Microsoft Entra

Microsoft rebranded Azure Active Directory to **Microsoft Entra ID** as part of consolidating its identity portfolio under the **Microsoft Entra** product family, which also includes:

```text
Microsoft Entra Family
        │
        ├── Microsoft Entra ID (core IAM — this note's focus)
        ├── Microsoft Entra ID Governance (entitlement mgmt, access reviews, PIM, lifecycle workflows)
        ├── Microsoft Entra Permissions Management (cloud infrastructure entitlement mgmt — CIEM)
        ├── Microsoft Entra Verified ID (decentralized identity / verifiable credentials)
        └── Microsoft Entra Global Secure Access
                  ├── Microsoft Entra Private Access (ZTNA for private apps)
                  └── Microsoft Entra Internet Access (SSE for internet/SaaS traffic)
```

> [!important] SC-300 tie-in
> Older study material and screenshots may still say "Azure AD" — the underlying concepts and portal locations are unchanged, just the branding. Don't let terminology alone throw you on a practice question.

---

# Entra ID vs. Active Directory Domain Services (AD DS)

|Aspect|AD DS (on-premises)|Microsoft Entra ID (cloud)|
|---|---|---|
|Structure|Hierarchical: domains, OUs, forests|Flat: single directory per tenant|
|Protocols|Kerberos, NTLM, LDAP|OAuth 2.0, OpenID Connect, SAML, WS-Fed|
|Management|Group Policy Objects (GPOs)|Conditional Access, device configuration policies|
|Trust model|Domain/forest trusts|Cross-tenant access settings, B2B federation|
|Typical use|Domain-joined Windows resources, legacy apps|Cloud apps, SaaS, modern auth, mobile/remote users|

Many organizations run both simultaneously — this is **hybrid identity**, covered in depth in [[Hybrid Identity]].

> [!important] SC-300 tie-in
> A recurring exam trap: assuming Entra ID has a direct equivalent to GPOs or OUs. It doesn't — the closest analogues are **Administrative Units** (for delegated scope, not policy) and **device configuration/Conditional Access policies** (for enforcement). Don't reach for AD DS mental models when answering Entra-specific questions.

---

# The Tenant Model

```text
Organization
      │
      ▼
Microsoft Entra Tenant
      │
      ├── Directory of Users, Groups, Devices, Apps
      ├── One or more verified custom domains
      ├── Associated Azure subscriptions (trust, not ownership)
      └── Associated Microsoft 365 subscriptions
```

A **tenant** is a dedicated, isolated instance of Entra ID created automatically when an organization signs up for a Microsoft cloud service. Each tenant has:

- A unique tenant ID (GUID)
- An initial default domain (`<tenantname>.onmicrosoft.com`)
- The ability to add and verify custom domains (e.g., `company.com`)

---

# Zero Trust and Identity

Entra ID is positioned as the enforcement point for Zero Trust's "never trust, always verify" principle. Every access decision considers:

```text
Identity
   +
Device
   +
Location / Network
   +
Application Sensitivity
   +
Risk Signal
   =
Access Decision
```

> [!important] SC-300 tie-in
> This model should feel familiar if you know Zscaler's Zero Trust architecture — the players are just renamed. Identity → Entra ID user/group; Device → device compliance/Conditional Access; Policy engine → Conditional Access; Enforcement → token issuance/blocking. The concepts transfer almost directly.

---

# Core Responsibilities of an Identity and Access Administrator

Per the exam's own audience profile:

- Configure and manage identities throughout their lifecycle (users, devices, Azure resources, applications)
- Implement authentication and authorization across apps and resources
- Troubleshoot, monitor, and report on identity and access
- Implement hybrid identity solutions
- Implement identity governance

---

# Related Notes

- [[Entra ID Editions and Licensing]]
- [[Core Terminology]]
- [[Tenant Configuration]]
- [[Entra Identities]]
- [[Hybrid Identity]]

# Entra ID Editions and Licensing

## Overview

Microsoft Entra ID is licensed in tiers, and many features referenced throughout the SC-300 exam — Conditional Access, PIM, Access Reviews, Identity Protection — are **gated by license edition**. Knowing which tier unlocks which feature is one of the highest-leverage things to memorize for this exam.

> [!important] SC-300 tie-in
> Exam scenario questions frequently test whether you know a feature exists in theory but is unavailable at the org's current license tier — the "correct" answer is often "recommend a license upgrade" rather than a configuration step.

Related:

- [[Entra ID Overview]]
- [[Conditional Access]]
- [[Privileged Identity Management]]
- [[Entitlement Management]]

---

# Core Editions

|Edition|Summary|
|---|---|
|**Free**|Included with any Microsoft cloud subscription (Azure, M365). Basic user/group management, cloud-only SSPR (admins only), basic reports|
|**P1**|Adds Conditional Access, hybrid identity features (Connect Health, self-service group/app management), SSPR with on-prem write-back, dynamic groups|
|**P2**|Everything in P1, plus Identity Protection, Privileged Identity Management (PIM), Access Reviews, and Entitlement Management|

```text
Free
  │
  ▼ (+ Conditional Access, hybrid write-back, dynamic groups)
P1
  │
  ▼ (+ Identity Protection, PIM, Access Reviews, Entitlement Management)
P2
```

> [!important] SC-300 tie-in
> The single most exam-relevant licensing fact: **Conditional Access requires at least P1**. Without it, tenant-wide legacy per-user MFA is the only enforcement option, which is a materially weaker and largely deprecated approach.

---

# Feature-to-License Quick Reference

|Feature|Minimum Edition|
|---|---|
|Basic user/group management|Free|
|Cloud-only SSPR (admins)|Free|
|Dynamic group membership|P1|
|SSPR with on-premises write-back|P1|
|Conditional Access|P1|
|Microsoft Entra Connect Health|P1|
|Identity Protection (risk-based policies)|P2|
|Privileged Identity Management (PIM)|P2|
|Access Reviews|P2|
|Entitlement Management|P2|

> [!important] SC-300 tie-in
> Custom security attributes and some newer Conditional Access controls (authentication context, protected actions) are also gated by P1/P2 in most configurations — if a scenario says an org is on Free tier and asks how to implement risk-based sign-in blocking, the real answer usually starts with licensing, not policy configuration.

---

# Microsoft Entra ID Governance

**Entra ID Governance** bundles the P2 governance capabilities — Entitlement Management, Access Reviews, PIM, and Lifecycle Workflows — into a licensing SKU that can be purchased as an add-on to P1, or is included with P2.

```text
Entra ID Governance
        │
        ├── Entitlement Management
        ├── Access Reviews
        ├── Privileged Identity Management (PIM)
        └── Lifecycle Workflows
```

---

# Microsoft Entra Suite

The **Microsoft Entra Suite** bundles Entra ID P2 together with the broader Entra product family:

```text
Microsoft Entra Suite
        │
        ├── Microsoft Entra ID P2
        ├── Microsoft Entra ID Governance
        ├── Microsoft Entra Verified ID
        ├── Microsoft Entra Private Access
        └── Microsoft Entra Internet Access
```

This is Microsoft's answer to a full Security Service Edge (SSE) + identity governance bundle, competing directly in the same space as Zscaler's broader platform.

> [!important] SC-300 tie-in
> [[Global Secure Access]] (Private Access + Internet Access) is licensed separately from core Entra ID P1/P2 — don't assume P2 alone unlocks GSA features on the exam.

---

# License Assignment Models

## Direct Assignment

Licenses assigned individually to a user object.

## Group-Based Licensing

Licenses assigned to a group; all members inherit the license automatically. Recommended at scale, since it removes manual per-user assignment and license-reconciliation drift.

```text
Group-Based Licensing

Security Group
      │
      ▼
License Assignment (once)
      │
      ▼
All Current & Future Members Inherit License
```

> [!important] SC-300 tie-in
> Group-based licensing can report assignment errors (e.g., conflicting service plans, insufficient available licenses) — know that these errors surface per-user in the Entra admin center and require investigation, since the exam may present a "user is missing an expected feature" scenario rooted in a licensing error rather than a policy misconfiguration.

---

# Related Notes

- [[Entra ID Overview]]
- [[Core Terminology]]
- [[Conditional Access]]
- [[Privileged Identity Management]]
- [[Entitlement Management]]
- [[Global Secure Access]]

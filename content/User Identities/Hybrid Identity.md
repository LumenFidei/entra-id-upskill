# Hybrid Identity

## Overview

**Hybrid identity** connects an on-premises Active Directory Domain Services (AD DS) environment to Microsoft Entra ID, so a single identity works across both. Most real-world enterprise Entra deployments are hybrid, not cloud-only — this is a heavily tested area.

> [!important] SC-300 tie-in
> The exam's audience profile explicitly calls out familiarity with AD DS as a prerequisite skill — expect hybrid identity questions to assume you understand both sides of the sync relationship, not just the cloud side.

Related:

- [[Core Terminology]]
- [[Entra Identities]]
- [[Authentication Methods]]

---

# Sync Options: Connect Sync vs. Cloud Sync

|Aspect|Microsoft Entra Connect Sync|Microsoft Entra Cloud Sync|
|---|---|---|
|Architecture|Full sync engine installed on-premises|Lightweight provisioning agent(s)|
|Multi-forest support|Supported, more complex configuration|Native, simpler multi-forest support|
|High availability|Requires staging server configuration|Multiple lightweight agents, easier HA|
|Customization|Deep sync rule customization possible|Limited customization, simpler by design|
|Best fit|Complex, single-purpose sync topologies|Multi-forest, simpler topologies, faster deployment|

```text
On-Premises AD DS
        │
        ├── Microsoft Entra Connect Sync (full sync engine)
        │         │
        │         ▼
        │    Microsoft Entra ID
        │
        └── Microsoft Entra Cloud Sync (lightweight agents)
                  │
                  ▼
            Microsoft Entra ID
```

> [!important] SC-300 tie-in
> Microsoft's current guidance favors **Cloud Sync** for new deployments and simpler topologies, reserving Connect Sync for scenarios needing its deeper customization (e.g., complex attribute filtering/transformation rules). If a scenario describes a brand-new, straightforward multi-forest deployment, Cloud Sync is usually the intended answer over Connect Sync.

---

# Authentication Methods for Hybrid Identity

## Password Hash Synchronization (PHS)

The user's password hash (technically a hash of the hash) is synchronized to Entra ID, allowing cloud-side authentication without a live connection back to on-premises AD DS at sign-in time.

```text
On-Prem AD DS
      │
      ▼
Password Hash Synced to Entra ID
      │
      ▼
User Authenticates Directly Against Entra ID
(no on-prem dependency at sign-in time)
```

## Pass-Through Authentication (PTA)

Sign-in requests are validated directly against on-premises AD DS in real time via a lightweight on-premises agent — the password itself never leaves the on-premises environment.

```text
User Signs In
      │
      ▼
Entra ID Receives Request
      │
      ▼
PTA Agent Validates Against On-Prem AD DS
      │
      ▼
Result Returned to Entra ID
```

> [!important] SC-300 tie-in
> The core PHS vs. PTA distinction the exam tests: PHS has **no on-premises dependency at sign-in time** (resilient to on-prem outages, since Entra ID can authenticate independently), while PTA **requires on-premises AD DS and the PTA agent to be available** for every sign-in. A scenario asking "which authentication method keeps working during an on-prem outage" is asking for PHS.

## Seamless Single Sign-On (SSO)

Complements PHS or PTA — provides silent sign-on for users on corporate-network, domain-joined devices via Kerberos, without a visible credential prompt.

## Federation (AD FS)

Legacy approach where authentication is fully delegated to an on-premises AD FS farm. Microsoft's current guidance is to **migrate away from AD FS federation** toward cloud authentication (PHS or PTA) where feasible, using a **staged rollout** to test the cloud auth experience for pilot groups before a full cutover.

```text
Legacy Federated Model

User Signs In
      │
      ▼
Redirected to On-Prem AD FS Farm
      │
      ▼
AD FS Validates Credentials
      │
      ▼
Token Issued Back to Entra ID
```

> [!important] SC-300 tie-in
> "Migrate from AD FS to other authentication and authorization mechanisms" is an explicit exam bullet. Know the staged rollout feature by name — it lets you move specific groups of users to PHS/PTA cloud authentication for testing while the rest of the org remains federated, without a disruptive all-or-nothing cutover.

---

# Microsoft Entra Connect Health

Monitoring service for hybrid identity infrastructure — tracks the health of Connect Sync servers, PTA agents, and (if still in use) AD FS servers.

```text
Connect Health Monitors
        │
        ├── Sync engine health & sync errors
        ├── PTA agent availability
        └── AD FS server & farm health (if federated)
```

> [!important] SC-300 tie-in
> Connect Health requires at least a P1 license. A scenario describing "no visibility into sync failures" on a Free-tier tenant again points to a licensing gap rather than a missing configuration step.

---

# Troubleshooting

## User Cannot Sign In After a Sync Change

Check:

1. Whether the identity is cloud-only vs. synced — synced identities have on-prem AD DS as their source of authority
2. Sync errors in Connect Health or the sync engine's own error log
3. Whether the authentication method (PHS/PTA) itself is healthy (PTA agent status, in particular)

---

## Users Report Slow or Failing Sign-In During On-Prem Maintenance

Check:

1. Authentication method in use — PTA and federation both depend on on-premises availability; PHS does not
2. PTA agent count and load distribution (single-agent deployments have no failover)

---

## Attribute Changes Not Reflecting in Entra ID

Check:

1. Whether the attribute is actually in scope for the sync rule (Connect Sync attribute filtering)
2. Sync cycle timing — changes aren't always instantaneous
3. Whether the change was made cloud-side on a synced object (source of authority conflict — see [[Core Terminology]])

---

# Best Practices

- Default to Cloud Sync for new, straightforward deployments; reserve Connect Sync for complex customization needs
- Prefer PHS over PTA/federation where business requirements allow, for resilience against on-premises outages
- Use staged rollout to de-risk any AD FS-to-cloud-authentication migration
- Monitor hybrid infrastructure proactively via Connect Health rather than reactively during an incident

---

# Related Notes

- [[Core Terminology]]
- [[Entra Identities]]
- [[Authentication Methods]]
- [[Tenant Configuration]]

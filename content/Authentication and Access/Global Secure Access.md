# Global Secure Access

## Overview

**Microsoft Entra Global Secure Access (GSA)** is Microsoft's Security Service Edge (SSE) offering — it extends identity-centric Zero Trust principles into network traffic itself, covering both private application access and general internet/SaaS traffic.

> [!note] Zscaler parallel
> If you've worked through the Zscaler ZDTE material, this entire feature area will feel structurally familiar. **Private Access** occupies roughly the same conceptual space as ZPA (private app access via connectors, no inbound exposure). **Internet Access** occupies roughly the same space as ZIA (cloud-delivered secure web gateway). The branding and console are different, but the architecture pattern — client agent, cloud broker, policy engine, no direct network exposure — is the same idea implemented by a different vendor.

> [!important] SC-300 tie-in
> Global Secure Access is licensed **separately** from core Entra ID P1/P2 (see [[Entra ID Editions and Licensing]]) — don't assume P2 alone unlocks it on the exam.

Related:

- [[Conditional Access]]
- [[Entra ID Editions and Licensing]]
- [[Identity Protection]]

---

# Components

```text
Microsoft Entra Global Secure Access
        │
        ├── Global Secure Access Client (endpoint agent)
        ├── Private Access (private app access — ZPA-shaped)
        ├── Internet Access (general internet/SaaS traffic — ZIA-shaped)
        └── Internet Access for Microsoft 365 (dedicated M365 traffic path)
```

---

# Global Secure Access Client

The endpoint agent that forwards eligible traffic into the Global Secure Access service, based on configured traffic forwarding profiles per traffic type (Private Access, Microsoft traffic, or general internet traffic).

```text
User Device
      │
      ▼
Global Secure Access Client
      │
      ├── Private Traffic ──────► Private Access
      ├── Microsoft 365 Traffic ─► Internet Access for M365
      └── General Internet Traffic ► Internet Access
```

> [!note] Zscaler parallel
> The GSA Client's traffic forwarding profile concept is directly analogous to a Zscaler Client Connector Forwarding Profile — same purpose (deciding what traffic to capture vs. bypass, per traffic category), different product.

---

# Private Access

Provides Zero Trust access to private, on-premises, or IaaS-hosted applications without a traditional VPN.

## Quick Access

Defines groups of private resources (by IP range, FQDN, or port range) that should be routed through Private Access — conceptually similar to a ZPA Application Segment, though typically coarser-grained (defined as address/port ranges rather than always per-application FQDNs).

## Private DNS

Resolves internal/private DNS names for resources accessed via Private Access, so private hostnames resolve correctly without needing a full VPN-based DNS configuration.

```text
User Requests Internal App
        │
        ▼
Private Access Evaluates Quick Access / App Segment Definition
        │
        ▼
Private DNS Resolves Internal Hostname
        │
        ▼
Traffic Routed via Global Secure Access (no inbound exposure)
```

> [!note] Zscaler parallel
> Private DNS here solves the exact same problem [[Application Segments]] and App Connector DNS resolution solve in ZPA — private hostnames need to resolve correctly for a client that isn't on the traditional corporate network.

---

# Internet Access

Provides cloud-delivered secure web gateway-style protection for general internet and third-party SaaS traffic — content filtering, threat protection, and policy enforcement, without backhauling traffic through on-premises infrastructure.

> [!note] Zscaler parallel
> This occupies the same role as ZIA's core secure web gateway function — URL/content filtering and threat protection applied at a cloud edge rather than on-premises appliances.

---

# Internet Access for Microsoft 365

A dedicated, optimized traffic path specifically for Microsoft 365 traffic — includes capabilities like **tenant restrictions** (ensuring users can only access your organization's Microsoft 365 tenant, not a personal or other-organization tenant) enforced at the network layer, complementing app-level Conditional Access controls.

```text
User Requests Microsoft 365
        │
        ▼
Internet Access for M365
        │
        ▼
Tenant Restriction Check (only approved tenant IDs allowed)
        │
        ▼
Optimized, Direct Path to Microsoft 365 Service
```

> [!important] SC-300 tie-in
> Tenant restrictions enforced via Internet Access for M365 close a gap that Conditional Access alone can't fully cover — CA governs access *into your own tenant*, but doesn't inherently stop a managed device from being used to sign into a *different* organization's tenant (e.g., a personal or competitor tenant) over the same network path.

---

# Conditional Access Integration

Global Secure Access traffic can itself become a **signal** for Conditional Access — for example, a "compliant network" condition that recognizes traffic verified as passing through Global Secure Access, allowing policies to treat that traffic similarly to how they might treat a trusted named location.

```text
Traffic Verified via Global Secure Access
        │
        ▼
"Compliant Network" Signal Available to Conditional Access
        │
        ▼
Conditional Access Policy Can Grant Reduced Friction
(e.g., skip an additional MFA prompt already covered by network assurance)
```

> [!important] SC-300 tie-in
> This network-as-a-signal integration is the key exam-relevant link between Global Secure Access and [[Conditional Access]] — know that GSA doesn't operate in isolation from the core identity policy engine; it feeds signal into it.

---

# Deployment

## Supported Platforms

Global Secure Access Client supports major desktop and mobile platforms, though feature parity (Private Access vs. Internet Access support) can vary by platform and is worth verifying against current documentation for any specific deployment.

## Deployment Steps (Conceptual)

```text
Enable Global Secure Access in Tenant
        │
        ▼
Deploy Global Secure Access Client to Endpoints
        │
        ▼
Configure Traffic Forwarding Profiles
        │
        ▼
Define Private Access Quick Access / App Segments (if using Private Access)
        │
        ▼
Configure Internet Access Policies (if using Internet Access)
        │
        ▼
Integrate Signals into Conditional Access
```

---

# Troubleshooting

## User Cannot Reach a Private Resource

Check:

1. Whether the traffic forwarding profile actually captures that traffic category (vs. bypassing it)
2. Quick Access / app segment definition covers the correct IP/FQDN/port range
3. Private DNS resolution for the target hostname

---

## User Accessing a Non-Organizational Microsoft 365 Tenant

Check:

1. Whether Internet Access for M365 tenant restrictions are actually configured and enforced
2. Whether the device's traffic is even being routed through Global Secure Access at all (client health/forwarding profile)

---

## Conditional Access Not Reflecting Expected Network Trust

Check:

1. Whether the "compliant network" signal integration is enabled and correctly referenced in the policy
2. Whether the device's traffic is verifiably passing through Global Secure Access (not bypassing it)

---

# Best Practices

- Treat Global Secure Access traffic forwarding profile design with the same care as any SSE client rollout — poorly scoped profiles cause the same "looks like a policy issue but is actually a forwarding issue" symptoms seen in other SSE platforms
- Use tenant restrictions via Internet Access for M365 as a deliberate, additional control layer — don't assume Conditional Access alone prevents access to unauthorized tenants
- Integrate the compliant network signal into Conditional Access deliberately, rather than treating GSA and CA as unrelated systems
- If you're coming from a Zscaler background, resist the urge to assume 1:1 feature parity — validate specific capabilities (like per-app segmentation granularity) against current Microsoft documentation rather than assuming ZPA-equivalent behavior everywhere

---

# Related Notes

- [[Conditional Access]]
- [[Entra ID Editions and Licensing]]
- [[Identity Protection]]

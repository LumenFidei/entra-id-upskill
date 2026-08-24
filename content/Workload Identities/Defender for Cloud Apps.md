# Defender for Cloud Apps

## Overview

**Microsoft Defender for Cloud Apps** is Microsoft's Cloud Access Security Broker (CASB) — it discovers shadow IT, assesses SaaS app risk, and applies real-time session controls, sitting downstream of Conditional Access as an additional enforcement and visibility layer for app access.

> [!note] Zscaler parallel
> This occupies the same conceptual space as a traditional CASB/SaaS Security capability — shadow IT discovery via traffic log analysis, sanctioning/unsanctioning apps, and applying controls to otherwise-uncontrolled SaaS usage. If you've worked with CASB-adjacent features elsewhere, the discovery-then-control workflow here will feel familiar even though the product and integration points differ.

> [!important] SC-300 tie-in
> This entire note maps to one exam bullet group: "Manage and monitor app access by using Microsoft Defender for Cloud Apps." Expect the exam to test whether you know **which specific Defender for Cloud Apps capability** solves a given problem, since several features here sound similar but serve different purposes.

Related:

- [[Conditional Access]]
- [[Enterprise Applications]]
- [[App Registrations]]

---

# Cloud Discovery

Analyzes traffic logs (from firewalls/proxies, or via Global Secure Access integration) to identify **all** cloud apps in use across the organization — including unsanctioned "shadow IT" apps IT never approved.

```text
Network/Proxy Traffic Logs
        │
        ▼
Cloud Discovery Analysis
        │
        ▼
Discovered Apps, Each Scored Against the Cloud App Catalog
        │
        ▼
Report: Sanctioned, Unsanctioned, and Unknown App Usage
```

> [!important] SC-300 tie-in
> Cloud Discovery answers **"what apps are actually being used"** — a fundamentally different question from Enterprise Applications, which only shows apps administrators have *already* explicitly integrated. A scenario describing "identify unauthorized SaaS usage the IT team doesn't know about" is asking for Cloud Discovery, not an Enterprise Applications report.

---

# Cloud App Catalog

A continuously updated, risk-scored database of thousands of SaaS applications (covering factors like compliance certifications, data handling practices, and known security posture), used to evaluate apps found via Cloud Discovery.

## Managing the Catalog

Administrators can manually **sanction** (approve) or **unsanction** (flag as disallowed) specific apps discovered in the environment, feeding into reporting and policy decisions.

```text
App Discovered
        │
        ▼
Scored Against Cloud App Catalog Risk Factors
        │
        ▼
Admin Sanctions or Unsanctions
        │
        ▼
Feeds Access/Session Policy Enforcement Decisions
```

---

# Connected Apps

Direct, API-based (not proxy-based) integrations with specific sanctioned SaaS platforms (e.g., major productivity or collaboration suites), providing deeper visibility and control than what's possible through network traffic inspection alone — such as scanning files at rest for sensitive content or auditing configuration settings within the app itself.

> [!important] SC-300 tie-in
> Connected apps use an **API connector** approach, which is architecturally different from Conditional Access App Control's reverse-proxy approach (below) — API connectors examine data *at rest* and app configuration, while the reverse proxy examines *sessions in real time*. A scenario emphasizing "scan existing files already stored in the SaaS app for sensitive data" points to a connected app / API connector, not session control.

---

# Application-Enforced Restrictions

Restrictions applied by the **target application itself**, triggered by a signal passed from Conditional Access (see [[Conditional Access]]) — for example, telling SharePoint Online to block downloads when a session originates from an unmanaged device.

```text
Conditional Access Session Control:
"App-Enforced Restrictions"
        │
        ▼
Signal Passed to Target App (e.g., SharePoint Online)
        │
        ▼
App Applies Its Own Native Restriction (e.g., Block Download)
```

---

# Conditional Access App Control

Routes a user's session through Defender for Cloud Apps as a **reverse proxy**, enabling real-time session monitoring and control for apps that don't natively support app-enforced restrictions — the session is intercepted, inspected, and controlled at the network layer rather than relying on the target app's own cooperation.

```text
User Signs In
        │
        ▼
Conditional Access Grants Access, Routes Session Through
Defender for Cloud Apps (Reverse Proxy)
        │
        ▼
Real-Time Session Monitoring/Control Applied
(e.g., block download, block copy/paste, block print)
        │
        ▼
Traffic Continues to the Actual SaaS App
```

> [!important] SC-300 tie-in
> **App-enforced restrictions** vs. **Conditional Access App Control** is a direct exam distinction: app-enforced restrictions require the target app to natively support the specific restriction being requested (limited app support), while Conditional Access App Control works for **any** app by intercepting the session itself (broader compatibility, since it doesn't depend on the app cooperating).

---

# Access and Session Policies

Configured within Defender for Cloud Apps to define what Conditional Access App Control actually does once a session is routed through it.

|Policy Type|Governs|
|---|---|
|Access policy|Whether access is granted or blocked at all, based on session context|
|Session policy|What the user can *do* within an already-granted session (block download, monitor only, require justification, etc.)|

```text
Access Policy
     │
     ▼
"Should this session be allowed at all?"

Session Policy
     │
     ▼
"Given the session is allowed, what actions are restricted within it?"
```

---

# OAuth App Policies

Governs the risk posed by **third-party OAuth apps** that end users have connected to sanctioned platforms (e.g., a productivity add-on requesting access to a user's mailbox or files) — distinct from your own organization's App Registrations, since these are typically external, user-consented apps.

```text
User Grants OAuth Consent to a Third-Party App
        │
        ▼
OAuth App Policy Evaluates Risk (permissions requested, publisher reputation)
        │
        ▼
Flagged for Review, Auto-Banned, or Allowed
```

> [!important] SC-300 tie-in
> OAuth app policies are the monitoring/governance layer that complements the **consent settings** covered in [[Enterprise Applications]] — consent settings control whether a user *can* grant access in the first place; OAuth app policies help you find and remediate risky grants that already happened (including ones granted before a stricter consent policy was put in place).

---

# Troubleshooting

## Unknown SaaS App Usage Discovered

Check:

1. Cloud Discovery logs/report for the specific app and its usage volume
2. Cloud App Catalog risk score for that app
3. Whether to sanction, unsanction, or actively block the app going forward

---

## Session Restriction Not Applying to a Specific App

Check:

1. Whether the app supports app-enforced restrictions natively (if attempting that route)
2. Whether Conditional Access is actually routing the session through App Control (reverse proxy) for that app
3. Session policy configuration and scope within Defender for Cloud Apps

---

## Risky Third-Party App Already Has User-Granted Access

Check:

1. OAuth app policies for that app's current risk classification
2. Whether it can be retroactively banned/revoked, not just blocked from future consent
3. Tenant consent settings, to prevent recurrence for other users

---

# Best Practices

- Run Cloud Discovery early and regularly — you can't govern shadow IT you don't know exists
- Use Conditional Access App Control for apps that don't support native app-enforced restrictions, rather than leaving those sessions uncontrolled
- Review OAuth app policies periodically, not just at initial rollout, since new risky grants can occur any time consent is permitted
- Treat sanctioning/unsanctioning in the Cloud App Catalog as a living process tied to actual discovered usage, not a one-time setup task

---

# Related Notes

- [[Conditional Access]]
- [[Enterprise Applications]]
- [[App Registrations]]
- [[Identity Protection]]

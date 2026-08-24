# Identity Protection

## Overview

**Microsoft Entra ID Protection** detects and responds to identity-based risk — compromised credentials, anomalous sign-ins, and risky workload identities — using Microsoft's threat intelligence and behavioral analytics.

> [!important] SC-300 tie-in
> Identity Protection requires a **P2** license (see [[Entra ID Editions and Licensing]]). A scenario describing risk-based sign-in blocking on a P1-only tenant has licensing as the actual gap, not a missing policy.

Related:

- [[Conditional Access]]
- [[Authentication Methods]]
- [[Entra ID Editions and Licensing]]
- [[Monitoring and Logs]]

---

# User Risk vs. Sign-In Risk

This distinction is the conceptual core of Identity Protection.

|Type|What It Measures|Example Detections|Evaluation Timing|
|---|---|---|---|
|**User risk**|Likelihood the user's *identity* (credentials) has been compromised|Leaked credentials found in a breach dump, anomalous user activity over time|Offline/batch, can surface after the fact|
|**Sign-in risk**|Likelihood a *specific sign-in attempt* is not the legitimate user|Impossible travel, anonymous IP, unfamiliar sign-in properties|Real-time, at the moment of sign-in|

```text
User Risk
   │
   ▼
"Is this identity likely compromised, in general?"

Sign-In Risk
   │
   ▼
"Is this specific sign-in attempt suspicious, right now?"
```

> [!important] SC-300 tie-in
> This is a near-certain exam item. A scenario describing "leaked credentials found on the dark web for a user" is **user risk**. A scenario describing "a single sign-in from an impossible-travel location" is **sign-in risk**. Picking the wrong one when configuring a risk-based Conditional Access policy is a classic distractor trap.

---

# Risk Levels

|Level|Typical Response|
|---|---|
|Low|Often monitored only, or lighter-touch response|
|Medium|Commonly triggers MFA requirement|
|High|Commonly triggers block or forced password reset|

---

# Remediation

## Self-Remediation

Users can resolve their own risk state by completing MFA or, for user risk, a self-service password reset — closing the loop without admin intervention, when policy allows it.

## Admin-Forced Remediation

Administrators can manually confirm a user compromised, dismiss risk, or force a password reset directly.

```text
Risk Detected
     │
     ▼
Risk-Based CA Policy Triggers Response
     │
     ├── User Self-Remediates (MFA / SSPR)
     │
     └── Admin Manually Investigates and Remediates
```

> [!important] SC-300 tie-in
> Risk-based responses are most commonly implemented **through Conditional Access policies** (using the user risk / sign-in risk conditions covered in [[Conditional Access]]), not through a separate standalone enforcement mechanism — Identity Protection detects and scores; Conditional Access (or, in some legacy configurations, Identity Protection's own built-in policies) is what acts on that score.

---

# MFA Registration Policy and Registration Campaigns

## MFA Registration Policy

Requires users to register for MFA within a grace period, closing the gap where an unregistered user has no strong authentication method available if a risk-based challenge fires.

## Registration Campaigns

Proactively nudges users (via Authenticator prompts) to move from a weaker registered method (like SMS) to a stronger one (like passwordless push or passkeys) — a rollout tool rather than an enforcement policy.

> [!important] SC-300 tie-in
> Registration campaigns are specifically named in the exam bullet list ("multifactor authentication registration by using authentication methods and registration campaigns") — treat this as its own testable concept, distinct from the MFA registration policy itself.

---

# Risky Workload Identities

Identity Protection also scores **service principals and applications** (not just human users) for risk — for example, a service principal exhibiting credential leaks or unusual application behavior patterns.

```text
Workload Identity (Service Principal / App)
        │
        ▼
Behavioral & Leak-Based Risk Detection
        │
        ▼
Risky Workload Identity Report
```

> [!important] SC-300 tie-in
> Risky workload identity detection connects directly to [[App and Azure Workload Identities]] in the Workload Identities domain — a scenario about a compromised application credential (not a human user) is testing this specific capability, and remediation typically means rotating the app's credential/secret, not resetting a "password" in the traditional sense.

---

# Monitoring: Risk Reports

|Report|Purpose|
|---|---|
|Risky users|Users currently flagged with an active risk state|
|Risky sign-ins|Individual sign-in events flagged as risky, with detail on the specific detection|
|Risk detections|Raw detection events feeding into the above two reports|

---

# Troubleshooting

## User Stuck in a Risky State Despite Completing MFA

Check:

1. Whether self-remediation is actually permitted by the applicable risk policy (some configurations require admin-only remediation)
2. Whether the user risk (not sign-in risk) requires a password change specifically, which MFA alone doesn't resolve

---

## Risk-Based Policy Not Triggering as Expected

Check:

1. License — confirm P2 is actually assigned/available
2. Whether the policy references user risk vs. sign-in risk correctly for the described scenario
3. Sign-in and risk detection logs for the actual detection that did (or didn't) fire

---

## Legitimate User Frequently Flagged as Risky

Check:

1. Travel patterns causing "impossible travel" false positives (e.g., VPN exit nodes in unusual locations)
2. Whether named/trusted locations are configured to reduce false positives for known corporate egress points

---

# Best Practices

- Implement risk-based Conditional Access policies rather than relying solely on static MFA requirements, for adaptive protection
- Use registration campaigns to proactively migrate users off weaker methods before disabling them outright
- Monitor risky workload identities with the same rigor as risky users — a compromised service principal can be just as damaging
- Review risk detection reports regularly, not only when a policy has already blocked someone

---

# Related Notes

- [[Conditional Access]]
- [[Authentication Methods]]
- [[Entra ID Editions and Licensing]]
- [[Monitoring and Logs]]

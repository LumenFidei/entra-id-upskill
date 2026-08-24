# Troubleshooting Methodology

## Overview

A structured approach to diagnosing Entra ID access problems by identifying **which layer** the failure actually lives in — identity, authentication, policy, device, application, or governance — before reaching for a fix. Most real (and exam-scenario) failures look like they belong to one layer but actually originate in another.

> [!important] SC-300 tie-in
> Nearly every troubleshooting scenario across this vault reduces to the same shape: a symptom presents at one layer, but the root cause sits at a different one. This note is the general model; [[Common Gotchas]] is the specific, memorized instance list.

Related:

- [[Common Gotchas]]
- [[Monitoring and Logs]]
- [[Conditional Access]]

---

# Troubleshooting Model

```text
Reported Problem
     │
     ▼
Identify Layer
     │
     ▼
Check the Logs for That Layer
     │
     ▼
Confirm or Rule Out
     │
     ▼
Move to Adjacent Layer if Ruled Out
     │
     ▼
Root Cause Identified
```

The discipline here is **not** jumping straight to the layer that feels most familiar — check the layer the symptom actually points to first, using logs rather than assumption.

---

# The Layers

## Layer 1: Identity / Directory State

Is the object (user, group, device, service principal) actually in the state you think it's in?

Check:

- Cloud-only vs. synced identity, and sync health ([[Hybrid Identity]])
- Usage location and license assignment state ([[Entra Identities]])
- Whether the object is inside a Restricted Management Administrative Unit ([[Tenant Configuration]])

---

## Layer 2: Authentication

Did the user actually prove their identity successfully, and with which method?

Check:

- Which authentication method was used vs. expected ([[Authentication Methods]])
- Whether MFA registration was actually completed
- Federation/PTA/PHS availability if on-premises dependency exists ([[Hybrid Identity]])

---

## Layer 3: Conditional Access / Policy

Did a policy grant, block, or add friction to this specific sign-in?

Check:

- Sign-in logs for exactly which CA policies were applied, satisfied, or failed ([[Conditional Access]])
- Report-only vs. enforced policy state
- The What If tool, for a direct simulation of the scenario

---

## Layer 4: Device

Does the device meet the posture/join requirements a policy demands?

Check:

- Compliant (Intune) vs. hybrid Azure AD joined vs. Entra joined state — these are not interchangeable ([[Core Terminology]], [[Conditional Access]])
- Device-enforced restriction application

---

## Layer 5: Application / Workload

Is the failure actually about the app's own configuration rather than the user at all?

Check:

- App Registration vs. Enterprise Application configuration split ([[App Registrations]], [[Enterprise Applications]])
- Client secret/certificate expiration
- Delegated vs. Application permission mismatch, and admin consent state
- Managed identity/service principal RBAC assignment, if it's a workload-to-workload failure ([[App and Azure Workload Identities]])

---

## Layer 6: Governance

Is access actually still valid on paper, even though it was granted correctly at some point?

Check:

- PIM eligible vs. active state ([[Privileged Identity Management]])
- Access review outcome and whether auto-apply actually removed access ([[Access Reviews]])
- Entitlement Management assignment expiration ([[Entitlement Management]])

---

# Evidence Collection

Before drawing a conclusion, gather:

- The exact user/object, application, and timestamp involved
- The specific error message or status code shown to the user
- Sign-in logs, audit logs, and (if relevant) provisioning logs for that window ([[Monitoring and Logs]])
- Whether the issue is reproducible, or was a one-time event

```text
Symptom
   │
   ▼
Evidence (logs, timestamps, exact error)
   │
   ▼
Hypothesis (which layer?)
   │
   ▼
Test Against That Layer's Logs
   │
   ▼
Root Cause
```

---

# Common Cross-Layer Failure Patterns

## "Looks Like a Policy Block But Isn't"

A user is denied access and the instinct is to blame Conditional Access — but the actual cause is an expired client secret on the app side, or a missing license on the user side. Always check sign-in logs first; they show which CA policies actually applied, which quickly rules CA in or out.

## "Looks Like the User's Problem But Is the App's"

A user reports "can't log into this one app." If other apps work fine for the same user, the failure is very likely at the application layer (App Registration/Enterprise Application config), not the identity or authentication layer.

## "Looks Like a Bug But Is Actually Expiration"

Access that "used to work" and now doesn't, with no reported change, is disproportionately often an **expiration** — a PIM eligible assignment lapsing back from active, an access package assignment expiring, or a client secret hitting its expiry date. Check expiration states before assuming a configuration regression.

---

# Root Cause Process

```text
Symptom Reported
     │
     ▼
Reproduce or Gather Evidence from Logs
     │
     ▼
Form a Hypothesis (which layer, which specific cause)
     │
     ▼
Test the Hypothesis Against That Layer's Data
     │
     ▼
Confirmed? ──── No ──── Move to Next Most Likely Layer
     │
    Yes
     │
     ▼
Remediate and Document
```

---

# Best Practices

- Start from logs, not instinct — sign-in logs alone resolve a large fraction of "was it CA or not" questions immediately
- Check expiration states (PIM, access packages, client secrets) before assuming a configuration change caused a regression
- When a single user reports an issue with a single app, suspect the application layer before the identity layer
- Use [[Common Gotchas]] as a fast-recall checklist once you've identified the likely layer, rather than starting the investigation from that list

---

# Related Notes

- [[Common Gotchas]]
- [[Monitoring and Logs]]
- [[Conditional Access]]
- [[Privileged Identity Management]]

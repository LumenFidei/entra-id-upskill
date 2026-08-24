# Conditional Access

## Overview

**Conditional Access (CA)** is Entra ID's policy engine — the mechanism that evaluates signals (identity, device, location, risk, application) at sign-in time and decides whether to grant, block, or restrict access. It is the single most heavily tested feature area on the SC-300 exam.

> [!important] SC-300 tie-in
> Conditional Access requires at least a P1 license (see [[Entra ID Editions and Licensing]]). Nearly every scenario question in this domain assumes CA is the mechanism — if an answer choice describes something CA can't do (like OU-based scoping), it's a distractor rooted in an AD DS mental model, not Entra ID.

Related:

- [[Authentication Methods]]
- [[Identity Protection]]
- [[Entra ID Editions and Licensing]]
- [[Global Secure Access]]

---

# Policy Anatomy

```text
Conditional Access Policy
        │
        ├── Assignments
        │      ├── Users and Groups (include/exclude)
        │      ├── Cloud Apps or Actions
        │      └── Conditions (risk, platform, location, client app)
        │
        └── Access Controls
               ├── Grant (require MFA, compliant device, app protection, etc.)
               └── Session (sign-in frequency, app-enforced restrictions, CA App Control)
```

A policy is fundamentally: **if these conditions are true for this user/app, then apply these controls.**

---

# Assignments

## Users and Groups

Include/exclude logic — always maintain at least one emergency access (break-glass) account excluded from restrictive policies. See [[Privileged Identity Management]] for break-glass account guidance.

## Cloud Apps or Actions

Target specific applications, or specific **user actions** (like registering security info) rather than an app at all.

## Conditions

|Condition|Example Use|
|---|---|
|Sign-in risk|Block or require MFA for medium/high risk sign-ins|
|User risk|Force password change for high-risk users|
|Device platform|Different requirements for iOS vs. Windows|
|Locations|Require MFA outside trusted/named locations|
|Client apps|Block legacy authentication protocols entirely|

> [!important] SC-300 tie-in
> Blocking **legacy authentication** (protocols that don't support modern auth/MFA, like older POP/IMAP/SMTP clients) via the client apps condition is one of the highest-value, most commonly recommended baseline policies — and a frequent exam answer for "how was MFA bypassed" scenarios, since legacy auth protocols historically couldn't be challenged for MFA at all.

---

# Grant Controls

|Control|Effect|
|---|---|
|Require MFA|Standard multifactor challenge|
|Require compliant device|Device must satisfy Intune compliance policy|
|Require hybrid Azure AD joined device|Device must be hybrid-joined (distinct from "compliant" — see [[Core Terminology]])|
|Require approved client app|Restricts to specific managed mobile apps|
|Require app protection policy|Enforces Intune App Protection (MAM) without full device enrollment|
|Require terms of use|User must accept a ToU document before proceeding|

Multiple grant controls can be combined with **require all** or **require one of the selected controls** logic.

> [!important] SC-300 tie-in
> "Require compliant device" and "require hybrid Azure AD joined device" are commonly confused as interchangeable — they are not. A device can be hybrid-joined but non-compliant (e.g., missing encryption), or Entra-joined (not hybrid) but fully compliant via Intune. Exam scenarios often hinge on picking the precisely correct control for the described device population.

---

# Session Controls

|Control|Effect|
|---|---|
|Sign-in frequency|Forces reauthentication after a defined period, overriding default token lifetime behavior|
|Persistent browser session|Controls whether browser sessions persist across browser restarts|
|App-enforced restrictions|Passes a signal to the app itself (e.g., SharePoint Online) to apply its own restrictions (like blocking downloads)|
|Conditional Access App Control|Routes session through Microsoft Defender for Cloud Apps as a reverse proxy for real-time session monitoring/control (see [[Defender for Cloud Apps]])|

---

# Policy Templates

Microsoft provides prebuilt Conditional Access policy templates in the Entra admin center covering common baseline scenarios (e.g., requiring MFA for admins, blocking legacy authentication, requiring compliant devices) — a faster starting point than building every policy from scratch.

> [!important] SC-300 tie-in
> Templates exist specifically to reduce time-to-value for common security baselines — a scenario emphasizing "quickly implement standard security recommendations" is pointing toward starting from a template rather than a fully custom policy build.

---

# Testing and Troubleshooting

## What If Tool

Simulates how existing CA policies would evaluate for a hypothetical (or real) user/app/condition combination, without actually attempting a sign-in.

## Report-Only Mode

Deploys a policy in a mode where it logs what **would** have happened without actually enforcing the grant/block — the standard, recommended way to validate a new policy before enforcing it in production.

```text
New Policy
     │
     ▼
Report-Only Mode
     │
     ▼
Review Sign-In Logs for Simulated Impact
     │
     ▼
Switch to "On" (Enforced)
```

> [!important] SC-300 tie-in
> Report-only mode is the textbook-correct answer whenever a scenario describes deploying a new, potentially disruptive CA policy safely — don't pick "enable the policy directly and monitor for complaints" as a best practice answer.

## Sign-In Logs

The authoritative source for diagnosing "why was/wasn't this user granted access" — shows exactly which CA policies were applied, satisfied, or failed for a given sign-in event.

---

# Continuous Access Evaluation (CAE)

Enables near real-time enforcement of certain critical events (user disabled, password changed, high user risk detected, network location change for supporting apps) — rather than waiting for the access token to naturally expire.

```text
Critical Event Occurs (e.g., user disabled)
        │
        ▼
CAE-Enabled Resource Revokes Access Near-Instantly
        │
        ▼
User Must Reauthenticate
```

> [!important] SC-300 tie-in
> CAE closes the "disabled account but token still valid for up to an hour" gap described in [[Authentication Methods]] — but only for **CAE-supported apps/scenarios**. Don't assume CAE covers every possible resource; the exam may test this boundary specifically.

---

# Authentication Context

Tags a **specific resource or action within an application** (not the whole app) so a more restrictive Conditional Access policy can apply only to that sensitive resource — for example, requiring a stronger control only when accessing a specific SharePoint site, rather than all of SharePoint Online.

```text
SharePoint Online (App-Level Policy: Standard MFA)
        │
        └── Specific "Finance Reports" Site (Authentication Context: Require Compliant Device)
```

> [!important] SC-300 tie-in
> Authentication context is the answer whenever a scenario needs **granular, resource-level** Conditional Access within a single application — not a separate app-level policy, which would be too broad.

---

# Protected Actions

Applies a step-up authentication requirement to specific **sensitive directory actions** themselves — for example, requiring re-authentication with a phishing-resistant method before an admin can modify Conditional Access policies, or before creating/updating role assignments.

```text
Admin Attempts Protected Action
(e.g., editing a Conditional Access policy)
        │
        ▼
Step-Up Authentication Challenge Enforced
        │
        ▼
Action Permitted Only After Satisfying Challenge
```

> [!important] SC-300 tie-in
> Protected actions defend against a scenario where an attacker has already compromised a session with a lower-assurance method — even with valid access, they'd be challenged again before touching specifically protected, high-impact settings. This is a newer capability and a good target for direct recall questions.

---

# Device-Enforced Restrictions

Restrictions enforced natively by the device/OS itself as directed by Conditional Access session controls (distinct from app-enforced restrictions, which rely on the target application's own cooperation).

---

# Troubleshooting

## Policy Not Applying as Expected

Check:

1. Assignment scope — included/excluded users, groups, and apps
2. Whether the policy is in Report-only vs. On state
3. The What If tool, to simulate the exact scenario in question
4. Sign-in logs for the specific user/time in question

---

## User Locked Out After New Policy Deployment

Check:

1. Whether the break-glass/emergency access account was excluded (should always be, but verify)
2. Whether the policy was deployed directly to "On" instead of validated in report-only mode first
3. Grant control logic — "require all" vs. "require one of" misconfiguration

---

## MFA Not Being Enforced Despite a CA Policy

Check:

1. Legacy authentication client apps bypassing modern auth entirely
2. Trusted/named location exclusions unintentionally covering the user's actual network
3. Conflicting or overlapping policies where a broader policy's grant logic supersedes intent

---

# Best Practices

- Always exclude break-glass accounts from Conditional Access policies that could lock out all administrators
- Validate every new policy in report-only mode before enforcing
- Block legacy authentication as a baseline policy in nearly every tenant
- Use authentication context for resource-level policy needs instead of over-broad app-level policies
- Use protected actions to defend the Conditional Access configuration surface itself, not just end-user resources

---

# Related Notes

- [[Authentication Methods]]
- [[Identity Protection]]
- [[Entra ID Editions and Licensing]]
- [[Global Secure Access]]
- [[Privileged Identity Management]]

# Common Gotchas

## Overview

This note consolidates the recurring "sounds similar but isn't," "silently breaks," and "commonly confused" patterns flagged throughout this vault — organized by SC-300 domain so review effort stays aligned with exam weighting. Each entry links back to its full explanation rather than repeating it in full here.

> [!important] SC-300 tie-in
> If you only have time to review one note before the exam, review this one — every entry here has already been flagged inline as a likely distractor pattern somewhere else in the vault.

---

# Domain 1: User Identities (20–25%)

|Gotcha|Why It Trips People Up|Full Note|
|---|---|---|
|Entra roles vs. Azure RBAC|A Global Administrator (Entra role) does **not** automatically manage Azure resources — that's a separate Azure RBAC grant|[[Core Terminology]]|
|Missing usage location blocks licensing|A user can exist and even sync correctly, but license assignment silently fails/blocks without usage location set|[[Entra Identities]]|
|Dynamic groups require P1|"Automatic membership by attribute" sounds like a config task, but on Free tier it's actually a licensing blocker|[[Entra ID Editions and Licensing]], [[Entra Identities]]|
|Custom security attributes need a dedicated role|Even Global Administrators can't manage these without being assigned Attribute Assignment/Definition Administrator|[[Entra Identities]]|
|Guest UPN `#EXT#` mismatch|ZPA/SAML apps expecting a UPN match fail with 401 unless the provisioning mapping and SAML claims are both corrected|[[External Identities]]|
|PHS vs. PTA resilience|PHS has no on-prem dependency at sign-in; PTA and federation both require on-prem availability for every sign-in|[[Hybrid Identity]]|
|AD FS migration is staged, not all-or-nothing|Staged rollout lets specific groups move to cloud auth before a full cutover — don't assume it's binary|[[Hybrid Identity]]|

---

# Domain 2: Authentication and Access (25–30%)

|Gotcha|Why It Trips People Up|Full Note|
|---|---|---|
|SMS/voice MFA vs. phishing-resistant methods|"Most secure" scenario language points to FIDO2/passkeys, CBA, or Windows Hello for Business — not SMS/voice, which are being phased down|[[Authentication Methods]]|
|Temporary Access Pass for passwordless onboarding|Bootstrapping a passwordless-only user shouldn't involve issuing a throwaway password — TAP is the named answer|[[Authentication Methods]]|
|Disabling an account ≠ revoking sessions|Active tokens can remain valid after disable unless sessions are explicitly revoked too|[[Authentication Methods]]|
|Compliant device vs. hybrid joined device|Two different grant controls — a device can be one without the other|[[Conditional Access]], [[Core Terminology]]|
|Legacy authentication bypasses MFA entirely|A common real "how was MFA bypassed" root cause — block legacy auth as a baseline policy|[[Conditional Access]]|
|Report-only mode before enforcement|The textbook-correct rollout step for any new, potentially disruptive CA policy|[[Conditional Access]]|
|CAE isn't universal|Continuous Access Evaluation only closes the token-validity gap for CAE-supported apps/scenarios, not everything|[[Conditional Access]]|
|User risk vs. sign-in risk|Leaked credentials = user risk (identity-level, offline). Impossible travel = sign-in risk (event-level, real-time)|[[Identity Protection]]|
|Risky workload identity ≠ password reset|Remediation for a compromised service principal means rotating its credential/secret, not resetting a "password"|[[Identity Protection]]|
|GSA is licensed separately from P1/P2|Don't assume Entra ID P2 alone unlocks Global Secure Access|[[Global Secure Access]], [[Entra ID Editions and Licensing]]|

---

# Domain 3: Workload Identities (20–25%)

|Gotcha|Why It Trips People Up|Full Note|
|---|---|---|
|App Registration vs. Enterprise Application|Same underlying object, two administrative lenses — secrets/redirect URIs live on the registration; user assignment/SSO config lives on the enterprise app|[[App Registrations]], [[Enterprise Applications]]|
|Delegated vs. Application permissions|A background service with no signed-in user **requires** Application permissions — Delegated permissions can't function without user context|[[App Registrations]]|
|Application permissions always need admin consent|There is no user-consent path for them, full stop — this is expected behavior, not a bug|[[App Registrations]]|
|Expired client secrets cause silent auth failures|"It was working and now it isn't, no code changed" is very often an expired secret|[[App Registrations]]|
|"User assignment required" defaulting off|A common root cause for unexpectedly broad app access — verify this toggle explicitly|[[Enterprise Applications]]|
|Cloud Application Administrator excludes Application Proxy|Only Application Administrator (or a custom role) can manage Application Proxy connectors|[[Enterprise Applications]]|
|App-enforced restrictions vs. CA App Control|App-enforced restrictions need the target app's native cooperation; CA App Control's reverse proxy works for any app|[[Defender for Cloud Apps]]|
|System-assigned managed identities can't convert to user-assigned|If multiple resources need to share one identity, you must create a user-assigned identity from scratch|[[App and Azure Workload Identities]]|
|Human accounts running services is an anti-pattern|Even if "it works," the correct recommendation is migrating to a managed identity or service principal|[[App and Azure Workload Identities]]|

---

# Domain 4: Identity Governance (20–25%)

|Gotcha|Why It Trips People Up|Full Note|
|---|---|---|
|Connected Organizations vs. Cross-Tenant Access Settings|CTAS governs trust/claims broadly; Connected Organizations enable self-service access-package discovery for a partner's users specifically|[[Entitlement Management]], [[External Identities]]|
|Access review auto-apply must be explicitly enabled|Without it, a completed review's decisions sit inert until manually applied|[[Access Reviews]]|
|PIM eligible vs. active|Effective permissions reflect only **active** assignments — a user can be eligible without current access|[[Privileged Identity Management]], [[Tenant Configuration]]|
|Break-glass accounts skip PIM activation on purpose|They must work even if PIM/CA itself has failed — requiring PIM activation for them is an anti-pattern, not a hardening step|[[Privileged Identity Management]]|
|Sign-in logs ≠ audit logs ≠ provisioning logs|A group membership change is an audit log question; a failed SCIM sync is a provisioning log question — sign-in logs cover neither|[[Monitoring and Logs]]|
|Log Analytics vs. storage account vs. Event Hub|Interactive querying, long-term compliance archival, and SIEM streaming are three different destinations, not interchangeable|[[Monitoring and Logs]]|

---

# Cross-Domain: Automation

|Gotcha|Why It Trips People Up|Full Note|
|---|---|---|
|`SigninLogs` only covers human interactive sign-ins|Investigating a service principal or managed identity requires `AADServicePrincipalSignInLogs` / `AADManagedIdentitySignInLogs` instead|[[KQL for Entra ID]]|
|Legacy AzureAD/MSOnline cmdlets are deprecated|Treat old-module syntax in an answer choice as a distractor signal — Microsoft Graph PowerShell SDK is current|[[PowerShell for Entra ID]], [[Entra Identities]]|
|Bulk scripts inherit the same attribute requirements as manual creation|A bulk-import script that skips usage location breaks licensing downstream just as surely as a manual omission would|[[PowerShell for Entra ID]]|

---

# How to Use This Note

Treat this as a pre-exam final pass, not a first-read introduction — each row assumes you've already read the linked note in full. If a row doesn't make immediate sense on its own, that's the signal to go back and read the source note rather than memorizing the row in isolation.

---

# Related Notes

- [[Troubleshooting Methodology]]
- [[Core Terminology]]
- [[Conditional Access]]
- [[Privileged Identity Management]]

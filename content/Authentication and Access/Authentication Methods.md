# Authentication Methods

## Overview

This note covers how users actually prove their identity to Entra ID: the modern authentication methods policy, MFA, SSPR, Windows Hello for Business, password protection, and Entra Kerberos for hybrid scenarios.

> [!important] SC-300 tie-in
> This is part of the largest exam domain (25–30%). Expect breadth here — the exam bullet list names specific methods (certificate-based auth, Temporary Access Pass, OAuth 2.0 tokens, Authenticator, passkeys) individually, suggesting each is fair game on its own, not just as a group.

Related:

- [[Hybrid Identity]]
- [[Conditional Access]]
- [[Identity Protection]]

---

# The Authentication Methods Policy

Modern, unified policy surface in the Entra admin center that replaces the older, separate "MFA service settings" and "SSPR authentication methods" configuration screens. Each method (SMS, Voice, Authenticator, FIDO2, CBA, etc.) is enabled/scoped independently, per user or group, from one place.

```text
Authentication Methods Policy
        │
        ├── Microsoft Authenticator
        ├── FIDO2 Security Key / Passkeys
        ├── Certificate-Based Authentication
        ├── Temporary Access Pass
        ├── SMS / Voice Call (legacy, being phased down)
        └── Password
```

> [!important] SC-300 tie-in
> Microsoft has been actively deprecating SMS and voice-call MFA in favor of phishing-resistant methods. If an exam scenario emphasizes "most secure" or "phishing-resistant" as the selection criterion, the intended answer is almost always **certificate-based authentication**, **FIDO2/passkeys**, or **Windows Hello for Business** — not SMS/voice.

---

# Certificate-Based Authentication (CBA)

Native, cloud-based authentication using X.509 client certificates — no AD FS or on-premises federation required. Configured via a Certificate Authority trust chain and username-binding rules (mapping certificate fields like SAN/UPN to a directory user).

```text
User Presents Client Certificate
        │
        ▼
Entra ID Validates Against Trusted CA
        │
        ▼
Certificate Field Mapped to Directory User (binding rule)
        │
        ▼
Access Granted
```

> [!important] SC-300 tie-in
> Know that CBA is **directly cloud-native** — this is a common exam distinction versus older certificate-based patterns that required AD FS. A scenario asking "how to support smart card users without AD FS" is describing CBA.

---

# Temporary Access Pass (TAP)

A time-limited, one-time or limited-use passcode that lets a user sign in **without** their normal credentials — most commonly used to bootstrap passwordless method registration (e.g., letting a new hire register a passkey or Authenticator without first needing a password).

```text
Admin Issues TAP
        │
        ▼
User Signs In With TAP (no password needed)
        │
        ▼
User Registers Passwordless Method (Authenticator, FIDO2, etc.)
        │
        ▼
Future Sign-Ins Use the New Passwordless Method
```

> [!important] SC-300 tie-in
> TAP is the answer whenever a scenario describes onboarding a user to passwordless authentication **without ever issuing them a traditional password** — this is a named, explicit exam bullet, not just a side detail.

---

# OAuth 2.0 Tokens

While full protocol-level OAuth implementation is more an app-registration/workload-identity topic, the administrator-relevant concepts are:

|Token Type|Purpose|
|---|---|
|Access Token|Grants access to a specific resource/API on behalf of the user or app|
|Refresh Token|Used to obtain a new access token without re-prompting the user|
|ID Token|Carries identity/authentication claims about the user (OpenID Connect)|

> [!important] SC-300 tie-in
> Session and token lifetime concepts here connect directly to [[Conditional Access]]'s Continuous Access Evaluation and sign-in frequency controls — a token being technically "valid" doesn't mean access is still permitted if a Conditional Access session control or CAE critical event has since revoked it.

---

# Microsoft Authenticator

Supports multiple modes:

- **Push notification approval** (with number matching for phishing resistance)
- **Passwordless phone sign-in** (Authenticator itself becomes the primary credential, no password step at all)
- **TOTP generation** (standard time-based one-time codes)

> [!important] SC-300 tie-in
> Number matching is enabled by default in current tenants and is a specific, named anti-phishing improvement over plain "Approve/Deny" push — expect this to be tested as the reason MFA fatigue attacks (a user blindly approving a push they didn't initiate) are mitigated.

---

# Passkeys (FIDO2)

Phishing-resistant, public-key based credentials. Two forms:

|Form|Description|
|---|---|
|Device-bound passkey (security key)|Hardware FIDO2 key, private key never leaves the device|
|Platform/synced passkey|Stored in a platform credential manager, can sync across a user's devices|

```text
FIDO2 Passkey Authentication

User Presents Passkey (biometric/PIN unlock)
        │
        ▼
Private Key Signs Challenge (never transmitted)
        │
        ▼
Entra ID Validates Signature Against Public Key
        │
        ▼
Access Granted
```

---

# Tenant-Wide MFA Settings

Legacy **per-user MFA** (enabled directly on individual user objects) still exists but is largely superseded by **Conditional Access-driven MFA**, which is more flexible (conditional on risk, location, app sensitivity, etc.) and is Microsoft's recommended approach.

> [!important] SC-300 tie-in
> A scenario that describes "MFA required for absolutely everyone, no exceptions, simplest possible setup" might still point to legacy per-user MFA or **Security Defaults** (a simplified, all-or-nothing baseline available even without P1) — but anything describing conditional or nuanced requirements should point you to Conditional Access instead. Don't over-index on Conditional Access being the answer to every MFA question.

---

# Self-Service Password Reset (SSPR)

Lets users reset or unlock their own accounts without helpdesk involvement, using registered authentication methods (phone, email, security questions, or Authenticator app).

## Combined Registration

A unified registration experience where users register their security info once, satisfying both SSPR and MFA method requirements together, rather than registering separately for each.

```text
Combined Registration
        │
        ├── SSPR Methods
        └── MFA Methods
              (shared, single registration flow)
```

> [!important] SC-300 tie-in
> SSPR with **on-premises write-back** (so a self-service reset in the cloud actually changes the on-prem AD DS password too) requires at least a P1 license — cloud-only SSPR is available even on Free tier, but only for cloud-only identities.

---

# Windows Hello for Business

Biometric or PIN-based sign-in backed by a TPM-protected key pair, replacing passwords for Windows devices.

## Deployment Models

|Model|Description|
|---|---|
|Cloud Kerberos Trust|Simplest, cloud-native trust model, minimal on-prem dependency|
|Hybrid Cloud Trust|For hybrid environments needing access to on-prem resources|
|Key Trust / Certificate Trust|Older on-premises PKI-dependent models|

> [!important] SC-300 tie-in
> **Cloud Kerberos Trust** is Microsoft's current recommended deployment model for most hybrid scenarios due to its reduced infrastructure requirements compared to older key/certificate trust models — if a scenario doesn't describe a specific legacy constraint, this is usually the intended answer.

---

# Disabling Accounts and Revoking Sessions

|Action|Effect|
|---|---|
|Block sign-in (disable account)|Prevents new authentication attempts|
|Revoke refresh tokens / sessions|Invalidates existing sessions, forcing re-authentication|

> [!important] SC-300 tie-in
> Disabling an account does **not** immediately terminate an already-active session — existing access/refresh tokens can remain valid until they expire unless sessions are explicitly revoked at the same time. A scenario describing "we disabled the account but the attacker still has access" is testing this exact gap — the correct remediation pairs disabling with a session/token revocation.

---

# Microsoft Entra Password Protection

- **Global banned password list** — Microsoft-maintained list of commonly-attacked passwords, enforced automatically
- **Custom banned password list** — org-defined additions (company name, product names, local terms)
- **On-premises extension** — proxy service + DC agents extend the same banned-password enforcement to on-premises AD DS password changes

```text
Password Change Request
        │
        ▼
Checked Against Global + Custom Banned List
        │
        ▼
Allowed or Rejected
```

---

# Microsoft Entra Kerberos for Hybrid Identities

Allows a hybrid Entra-joined device to obtain a Kerberos ticket **issued by Entra ID** (via a lightweight Kerberos Server object published to AD DS) to access on-premises resources — without needing a direct line-of-sight to a traditional on-prem domain controller for that ticket issuance step.

```text
Hybrid Entra-Joined Device
        │
        ▼
Entra ID Issues Kerberos Ticket
        │
        ▼
Device Accesses On-Prem Resource Using the Ticket
```

> [!important] SC-300 tie-in
> This feature specifically supports **Windows Hello for Business Cloud Kerberos Trust** deployments and modern remote-work scenarios where a device may rarely (or never) directly contact an on-prem DC — know it as the mechanism that makes Cloud Kerberos Trust practical for remote users.

---

# Troubleshooting

## User Stuck Unable to Register MFA Method

Check:

1. Whether a Temporary Access Pass is needed to bootstrap registration for a passwordless-only rollout
2. Authentication methods policy scope — is the intended method actually enabled for this user/group

---

## Disabled Account Still Shows Active Sessions

Check:

1. Whether session/token revocation was performed in addition to disabling the account
2. Continuous Access Evaluation status — CAE can shorten this gap for supported critical events, but isn't universal for every disable action

---

## Password Reset Not Reflected On-Premises

Check:

1. Whether SSPR write-back is licensed (P1+) and enabled
2. On-premises write-back service/agent health

---

# Best Practices

- Prioritize phishing-resistant methods (FIDO2/passkeys, CBA, Windows Hello for Business) over SMS/voice wherever feasible
- Use Temporary Access Pass for passwordless onboarding rather than issuing a throwaway initial password
- Pair account disablement with explicit session revocation as standard offboarding/incident procedure
- Default to Cloud Kerberos Trust for new Windows Hello for Business hybrid deployments

---

# Related Notes

- [[Hybrid Identity]]
- [[Conditional Access]]
- [[Identity Protection]]
- [[Entra ID Editions and Licensing]]

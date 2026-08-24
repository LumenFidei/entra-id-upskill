# Enterprise Applications

## Overview

**Enterprise Applications** is the administrative surface for managing a service principal within your tenant — user/group assignment, SSO configuration, provisioning, and the settings that govern how an application actually behaves for your organization's users. It's the operational counterpart to [[App Registrations]], which defines the app's identity itself.

> [!important] SC-300 tie-in
> If a scenario is about *assigning users*, *configuring SSO for a specific app instance*, or *reviewing what an app can already do in your tenant*, it's an Enterprise Applications question — not an App Registration one.

Related:

- [[App Registrations]]
- [[App and Azure Workload Identities]]
- [[Defender for Cloud Apps]]
- [[Tenant Configuration]]

---

# Application-Level vs. Tenant-Level Settings

|Scope|Examples|
|---|---|
|Application-level|SSO configuration, user/group assignment, app-specific consent, provisioning mapping for that one app|
|Tenant-level|Whether users can consent to apps at all, default user permissions to view/register apps, overall enterprise app visibility settings|

```text
Tenant-Level Settings
        │
        ▼ (govern the boundaries within which...)
Application-Level Settings
        │
        ▼
Individual App Behavior for Your Org's Users
```

> [!important] SC-300 tie-in
> A scenario describing "no users anywhere in the org can consent to any new app" is a **tenant-level** setting problem. A scenario describing "this one specific app isn't requiring MFA/SSO correctly" is **application-level**. Confusing the two scopes is a common distractor pattern.

---

# Assigning Entra Roles to Manage Enterprise Applications

Rather than granting Global Administrator for app management tasks, the built-in **Application Administrator** and **Cloud Application Administrator** roles are purpose-built for this.

|Role|Scope|
|---|---|
|Application Administrator|Full management of all app registrations and enterprise apps|
|Cloud Application Administrator|Same as Application Administrator, but **excludes** Application Proxy configuration|

> [!important] SC-300 tie-in
> The Application Administrator vs. Cloud Application Administrator distinction is specifically about **Application Proxy** — if a scenario needs someone to manage Application Proxy connectors and on-prem app publishing, Cloud Application Administrator is **not sufficient**; it must be Application Administrator (or a custom role with that permission).

---

# Microsoft Entra Application Proxy

Publishes on-premises web applications for secure remote access **without a VPN**, using a lightweight connector installed on-premises that establishes an outbound-only connection to the Entra ID cloud service.

```text
Remote User
      │
      ▼
Microsoft Entra ID (Application Proxy)
      │
      ▼
Application Proxy Connector (on-premises, outbound-only)
      │
      ▼
Internal Web Application
```

> [!note] Zscaler parallel
> This is conceptually the same shape as ZPA's App Connector model — an outbound-only, on-premises connector removing the need for inbound firewall rules or VPN, brokered by a cloud service. The granularity and feature set differ, but the "no inbound exposure, cloud-brokered access" pattern is identical in spirit.

> [!important] SC-300 tie-in
> Application Proxy is specifically for **on-premises web applications** — it's the named exam answer whenever a scenario describes "publish an internal web app for remote users without a VPN." Don't confuse it with Enterprise Applications' general SaaS integration (covered next), which is for third-party cloud apps, not your own on-prem apps.

---

# Integrating SaaS Applications

For third-party SaaS apps, integration typically means:

```text
Add from Gallery (Pre-Integrated) or Create Non-Gallery App
        │
        ▼
Configure Single Sign-On (SAML, OIDC, or password-based)
        │
        ▼
Configure Automatic User Provisioning (SCIM, if supported)
        │
        ▼
Assign Users/Groups
```

Many popular SaaS apps have pre-built gallery entries with guided SSO/provisioning setup; less common or custom apps use a non-gallery (generic SAML/OIDC) integration path.

---

# Assigning, Classifying, and Managing Users, Groups, and App Roles

Enterprise Applications can require explicit user/group assignment before anyone can sign in (a security-relevant toggle — "user assignment required") and can use [[App Registrations|App Roles]] to grant differentiated access within the app for different assigned users/groups.

> [!important] SC-300 tie-in
> "User assignment required" defaulting to **off** for apps added via certain flows is a common misconfiguration the exam may test — with it off, *any* user in the tenant can sign into the app (subject to Conditional Access), not just explicitly assigned ones. A scenario describing unexpected/unauthorized app access often traces back to this toggle.

---

# Configuring and Managing User and Admin Consent

Tenant-wide consent settings determine whether users can consent to apps themselves, whether they can consent only to apps from verified publishers with limited permission risk, or whether all consent must go through an administrator.

```text
Consent Settings Spectrum

Do not allow user consent (admin consent required for everything)
        │
Allow user consent for apps from verified publishers, for permissions classified as low risk
        │
Allow user consent for all apps
```

Administrators can also review and act on **user consent requests** when self-service consent is restricted, and monitor **risky consent** patterns (e.g., a burst of similar consent requests across users, indicative of a phishing/consent-grant attack).

> [!important] SC-300 tie-in
> "Consent phishing" — tricking a user into granting a malicious app permissions via a legitimate-looking consent prompt — is a real attack pattern Microsoft's tooling actively defends against. The middle consent setting (verified publisher + low-risk permissions only) is Microsoft's commonly recommended balance between usability and this specific risk, and a good default answer when a scenario asks for a security-conscious but not maximally restrictive consent policy.

---

# Application Collections

Logical groupings of enterprise applications, used to organize how apps appear to end users in the My Apps portal — for example, grouping all Finance-department SaaS apps together for easier discovery by Finance users.

```text
Application Collection: "Finance Apps"
        │
        ├── Expense Reporting SaaS App
        ├── Payroll SaaS App
        └── Budgeting SaaS App
        │
        ▼
Surfaced Together in My Apps for Assigned Finance Users
```

> [!important] SC-300 tie-in
> Application collections are a **user-experience/organization feature**, not a security or access-control mechanism — don't confuse them with Administrative Units or access packages ([[Entitlement Management]]), which do have real access-scoping implications.

---

# Troubleshooting

## Users Unexpectedly Able to Sign Into an App

Check:

1. "User assignment required" toggle status on the Enterprise Application
2. Whether an overly broad group is assigned to the app
3. Applicable Conditional Access policies scoping (or failing to scope) that app

---

## On-Premises App Unreachable via Application Proxy

Check:

1. Application Proxy connector health/status (on-premises)
2. Outbound connectivity from the connector to the Entra ID cloud service
3. Internal URL configuration matching the actual on-premises app address

---

## Users Cannot Consent to a Needed App

Check:

1. Tenant-wide consent policy setting (may require admin consent only)
2. Whether the app is requesting Application permissions (never user-consentable, per [[App Registrations]])
3. Whether the app has been flagged by risk-based consent controls, requiring explicit admin review

---

# Best Practices

- Require explicit user assignment for any application handling sensitive data, rather than leaving it open to all tenant users
- Use Application Administrator (not Cloud Application Administrator) only for admins who genuinely need to manage Application Proxy
- Set tenant consent policy to at least "verified publishers, low-risk permissions only" rather than unrestricted user consent
- Use application collections to improve discoverability, but never rely on them as an access-control boundary

---

# Related Notes

- [[App Registrations]]
- [[App and Azure Workload Identities]]
- [[Defender for Cloud Apps]]
- [[Tenant Configuration]]

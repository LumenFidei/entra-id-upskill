# Entitlement Management

## Overview

**Entitlement Management** automates the request, approval, assignment, and time-bound expiration of access to groups, applications, and SharePoint sites — bundled into a single requestable unit called an **access package**. It's the primary tool for scaling access governance beyond manual, ad-hoc group additions.

> [!important] SC-300 tie-in
> Entitlement Management requires a **P2** license (see [[Entra ID Editions and Licensing]]). A scenario describing automated, time-bound access requests on a P1-only tenant has licensing as the real blocker.

Related:

- [[Access Reviews]]
- [[External Identities]]
- [[Entra ID Editions and Licensing]]
- [[Conditional Access]]

---

# Core Hierarchy

```text
Catalog
   │
   ▼
Access Package(s)
   │
   ▼
Policy (who can request, approval flow, expiration)
   │
   ▼
Resource Roles Granted
(groups, Teams, apps, SharePoint sites)
```

---

# Catalogs

A **catalog** is a container of resources (groups, apps, SharePoint sites) that can be bundled together into access packages. Catalogs are typically organized around a business function or project.

```text
Catalog: "Finance Project X"
        │
        ├── Group: Finance-ProjectX-Team
        ├── App: Budgeting SaaS App
        └── SharePoint Site: ProjectX Documents
```

Catalogs can be scoped to allow specific **resource owners** (not just global admins) to add their own resources into the catalog for packaging — enabling delegated, business-owner-driven governance rather than centralized IT bottlenecks.

> [!important] SC-300 tie-in
> Delegating catalog and access package creation to resource owners (not IT admins) is a deliberately emphasized design goal of Entitlement Management — a scenario describing "business owners should manage their own project access without filing IT tickets" is describing this delegation model.

---

# Access Packages

The requestable bundle of resource roles a user actually asks for.

```text
Access Package: "New Contractor Onboarding"
        │
        ├── Group Membership: Contractors-General
        ├── App Assignment: Timesheet App (Reader role)
        └── SharePoint Site Access: Contractor Docs (Read)
```

## Access Package Policies

Each access package has one or more **policies** governing:

- Who can request it (internal users, specific connected organizations, or all external users)
- Approval requirements (single/multi-stage approval, specific approvers or manager-based approval)
- Access duration and expiration (fixed date, or a defined duration with optional renewal/re-request)

```text
User Requests Access Package
        │
        ▼
Policy Evaluates Eligibility to Request
        │
        ▼
Approval Stage(s) (if configured)
        │
        ▼
Access Granted for Defined Duration
        │
        ▼
Access Expires Automatically (unless renewed)
```

> [!important] SC-300 tie-in
> Automatic expiration is a core value proposition tested on the exam — entitlement management access is **time-bound by design**, directly addressing "access creep" (users accumulating permissions indefinitely) in a way manual group management does not.

---

# Managing Access Requests

Administrators (or delegated approvers) can view pending requests, approve/deny with justification, and audit historical request activity — including requests initiated by external users through connected organizations.

---

# Terms of Use (ToU)

A ToU is a document (e.g., a PDF) users must explicitly accept before being granted access — usable standalone or as a requirement within a Conditional Access policy grant control (see [[Conditional Access]]).

```text
Access Attempt
      │
      ▼
Terms of Use Presented
      │
      ▼
User Must Accept Before Proceeding
      │
      ▼
Acceptance Logged (with version, timestamp)
```

> [!important] SC-300 tie-in
> ToU acceptance is versioned and auditable — if a document is updated, users can be required to re-accept the new version. A scenario describing "prove which users agreed to the updated privacy policy" is asking about ToU version tracking and reporting, not a general audit log.

---

# Managing the Lifecycle of External Users

Entitlement management can tie a guest user's **entire lifecycle** — from initial access package request through to automatic removal — to their actual need for access, including automatically **removing the guest account entirely** once all their access packages expire and aren't renewed.

```text
External User Requests Access Package
        │
        ▼
Guest Account Created (if not already existing)
        │
        ▼
Access Granted for Defined Duration
        │
        ▼
Access Expires
        │
        ▼
(Optional) Guest Account Automatically Removed
if No Other Access Remains
```

> [!important] SC-300 tie-in
> This automatic guest cleanup directly addresses "orphaned guest accounts" — a common real-world audit finding where external users retain tenant access long after a project ended. A scenario describing stale guest accounts as a governance problem is pointing toward entitlement management's lifecycle automation, not a manual cleanup script.

---

# Connected Organizations

A **connected organization** represents a specific external partner tenant (or a non-Entra domain) whose users are pre-authorized to discover and request access packages — without needing individual invitations sent in advance, since discoverability is scoped at the organization level rather than the individual user level.

```text
Connected Organization: "Partner Corp"
        │
        ▼
Any User from Partner Corp's Tenant Can Discover
and Request Applicable Access Packages
        │
        ▼
Guest Account Created On-Demand Upon First Approved Request
```

> [!important] SC-300 tie-in
> Connected organizations differ from the [[External Identities]] cross-tenant access settings — cross-tenant access settings govern **trust and claims** between tenants broadly, while connected organizations specifically enable **self-service discovery and request** of access packages for users from that partner, without requiring them to already exist as guests beforehand.

---

# Troubleshooting

## Access Not Automatically Expiring as Expected

Check:

1. Access package policy expiration/duration settings
2. Whether the user renewed access before expiration (renewal resets the clock)
3. Whether the assignment was made outside entitlement management entirely (e.g., manual group addition bypassing the package)

---

## External User Cannot Discover an Access Package

Check:

1. Whether their organization is configured as a connected organization
2. Access package policy scope — is it open to "all external users" vs. restricted to specific connected organizations
3. Catalog visibility/permissions for that access package

---

## Guest Account Not Removed After Access Expired

Check:

1. Whether the "remove guest when access expires" lifecycle setting is actually enabled on the relevant policy
2. Whether the guest still holds *other*, unrelated access outside of entitlement management (which would correctly prevent automatic removal)

---

# Best Practices

- Delegate catalog and access package ownership to business resource owners rather than centralizing everything in IT
- Set explicit expiration/renewal periods on every access package — avoid indefinite access as a default
- Use connected organizations for recurring, ongoing partner relationships rather than repeatedly re-inviting the same external users
- Enable automatic guest account removal tied to access package expiration, to prevent orphaned guest accounts from accumulating

---

# Related Notes

- [[Access Reviews]]
- [[External Identities]]
- [[Entra ID Editions and Licensing]]
- [[Conditional Access]]
- [[Privileged Identity Management]]

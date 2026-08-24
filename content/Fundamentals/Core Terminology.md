# Core Terminology

## Overview

A shared vocabulary of Entra ID objects, roles, and identity types underpins every other note in this vault. Getting these definitions precise — especially the ones that sound similar but aren't — pays off across all four exam domains.

> [!important] SC-300 tie-in
> Precision matters more than familiarity here. Several terms below (Entra roles vs. Azure RBAC, Entra joined vs. Entra registered) are near-guaranteed sources of distractor answers on the exam.

Related:

- [[Entra ID Overview]]
- [[Tenant Configuration]]
- [[Entra Identities]]

---

# Directory Object Types

```text
Microsoft Entra Directory
        │
        ├── Users
        ├── Groups
        ├── Devices
        ├── Applications (App Registrations)
        ├── Service Principals
        └── Administrative Units
```

|Object|What It Represents|
|---|---|
|User|A person (or workload, if service account-style) with sign-in credentials|
|Group|A collection of users/devices/other groups for access assignment|
|Device|A registered or joined endpoint (Windows, macOS, iOS, Android, Linux)|
|Application (App Registration)|The definition of an app's identity in Entra ID — its "blueprint"|
|Service Principal|The local, usable instance of an application within a specific tenant|
|Administrative Unit|A container used to scope administrative permissions to a subset of the directory|

---

# Microsoft Entra Roles vs. Azure RBAC

This is one of the most consistently confused distinctions for people new to Entra ID — including experienced infrastructure engineers.

|Aspect|Microsoft Entra Roles|Azure RBAC|
|---|---|---|
|Controls access to|The **identity plane** — directory objects, Entra ID settings, other administrators|The **resource plane** — Azure resources (VMs, storage, networking, subscriptions)|
|Example role|Global Administrator, User Administrator, Conditional Access Administrator|Owner, Contributor, Reader, Virtual Machine Contributor|
|Scope|Directory-wide or Administrative Unit|Management group, subscription, resource group, or individual resource|
|Assigned via|Entra admin center → Roles and administrators|Azure portal → Access control (IAM) on a resource|

```text
Microsoft Entra Roles                    Azure RBAC
        │                                     │
        ▼                                     ▼
  Directory Objects                    Azure Resources
(users, groups, apps,                 (VMs, storage accounts,
 Entra ID settings)                    resource groups, subscriptions)
```

> [!important] SC-300 tie-in
> This is a near-certain exam item. A "Global Administrator" (Entra role) does **not** automatically have access to manage Azure resources like virtual machines — that requires a separate Azure RBAC role assignment (though a Global Admin can elevate themselves to User Access Administrator at the root management group scope to grant themselves that access).

---

# Identity Types

|Type|Description|
|---|---|
|Cloud-only|Created and managed entirely in Entra ID; no on-premises AD DS counterpart|
|Synced (Hybrid)|Originates in on-premises AD DS, synchronized to Entra ID via Connect Sync or Cloud Sync|
|External / Guest|An identity from another organization or consumer identity provider, invited into the tenant (see [[External Identities]])|

> [!important] SC-300 tie-in
> Synced identities have their **source of authority** in on-premises AD DS — meaning most attribute changes (and sometimes password changes, depending on the authentication method) must happen on-premises and sync up, not directly in the cloud. This trips people up in troubleshooting scenarios where an admin tries to edit a synced user's attribute directly in the Entra admin center and it silently reverts.

---

# Device Join States

|State|Description|Typical Scenario|
|---|---|---|
|Microsoft Entra Registered|Personal/BYOD device registers identity without full management|BYOD, personal phones/laptops|
|Microsoft Entra Joined|Cloud-native device fully joined to Entra ID, no on-prem AD DS account|Cloud-first/modern-managed corporate devices|
|Microsoft Entra Hybrid Joined|Device joined to both on-prem AD DS and Entra ID|Traditional domain-joined corporate devices transitioning to cloud|

```text
Personal Device ──── Entra Registered
Cloud-Native Corp Device ──── Entra Joined
Traditional Domain Device ──── Entra Hybrid Joined
```

> [!important] SC-300 tie-in
> Conditional Access "require compliant device" and "require hybrid Azure AD joined device" are **different grant controls** that overlap in intent but aren't interchangeable — a hybrid-joined device is not automatically "compliant" unless Intune/co-management also marks it so.

---

# Administrative Units (Preview Term Here, Detailed in Tenant Configuration)

A container object used to delegate administrative scope to a subset of the directory (e.g., "only manage users in the Sales department") without granting tenant-wide role permissions. Covered in full in [[Tenant Configuration]].

---

# Related Notes

- [[Entra ID Overview]]
- [[Tenant Configuration]]
- [[Entra Identities]]
- [[External Identities]]
- [[Hybrid Identity]]

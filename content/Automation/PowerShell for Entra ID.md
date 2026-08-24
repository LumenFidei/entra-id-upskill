# PowerShell for Entra ID

## Overview

The **Microsoft Graph PowerShell SDK** is the current, supported way to script Entra ID administration — it replaced the older, now-deprecated **AzureAD** and **MSOnline** modules. Since it's a wrapper around [[Microsoft Graph API]], every cmdlet maps directly to an underlying Graph REST call.

> [!important] SC-300 tie-in
> If an exam question presents cmdlet syntax from the legacy `AzureAD` or `MSOnline` modules as an answer choice, treat it as a strong distractor signal — Microsoft's current guidance is Microsoft Graph PowerShell SDK for all new scripting.

Related:

- [[Microsoft Graph API]]
- [[Entra Identities]]
- [[KQL for Entra ID]]

---

# Installing and Connecting

```powershell
# Install (one-time)
Install-Module Microsoft.Graph -Scope CurrentUser

# Connect, requesting only the scopes actually needed
Connect-MgGraph -Scopes "User.ReadWrite.All", "Group.ReadWrite.All"
```

```text
Connect-MgGraph -Scopes "..."
        │
        ▼
Interactive Sign-In (Delegated Permissions)
        │
        ▼
Session Authorized for Requested Scopes Only
```

> [!important] SC-300 tie-in
> `Connect-MgGraph -Scopes` is a **delegated permission** request, tied to the signed-in admin's own consent and existing privileges — see [[App Registrations]] and [[Microsoft Graph API]] for the delegated vs. application distinction. An interactive PowerShell session cannot exceed what the connecting user is already permitted to do or consent to.

---

# Modular Structure

The SDK is split into many submodules by functional area rather than one giant monolithic module — installing/importing the right submodule matters.

```text
Microsoft.Graph (root module)
        │
        ├── Microsoft.Graph.Users
        ├── Microsoft.Graph.Groups
        ├── Microsoft.Graph.Identity.SignIns   (Conditional Access, Identity Protection)
        ├── Microsoft.Graph.Identity.Governance (Entitlement Management, Access Reviews, PIM)
        └── Microsoft.Graph.Reports             (sign-in/audit log reporting cmdlets)
```

> [!important] SC-300 tie-in
> A scenario describing "the cmdlet I expect isn't available" is often a missing/unimported submodule rather than a syntax error — PIM- and governance-related cmdlets specifically live under `Microsoft.Graph.Identity.Governance`, not the core `Users`/`Groups` submodules.

---

# Common Cmdlets by Domain

## Users and Groups ([[Entra Identities]])

```powershell
New-MgUser -DisplayName "Jane Doe" -UserPrincipalName "jane.doe@contoso.com" ...
Get-MgUser -UserId "jane.doe@contoso.com"
Update-MgUser -UserId "jane.doe@contoso.com" -UsageLocation "US"
New-MgGroup -DisplayName "Sales Team" -MailEnabled:$false -SecurityEnabled:$true ...
New-MgGroupMember -GroupId $groupId -DirectoryObjectId $userId
```

## Sign-In and Audit Reporting ([[Monitoring and Logs]])

```powershell
Get-MgAuditLogSignIn -Filter "userPrincipalName eq 'jane.doe@contoso.com'"
Get-MgAuditLogDirectoryAudit -Filter "activityDisplayName eq 'Add member to role'"
```

## Privileged Identity Management ([[Privileged Identity Management]])

```powershell
Get-MgRoleManagementDirectoryRoleAssignmentScheduleRequest
New-MgRoleManagementDirectoryRoleAssignmentScheduleRequest -Action "selfActivate" ...
```

## Entitlement Management ([[Entitlement Management]])

```powershell
New-MgEntitlementManagementAccessPackage -DisplayName "New Contractor Bundle" ...
New-MgEntitlementManagementAssignmentRequest -AccessPackageId $packageId ...
```

> [!important] SC-300 tie-in
> Notice the cmdlet naming convention: `Mg` + functional area + noun. Recognizing this pattern helps you infer what a cmdlet does even if you haven't memorized it exactly — a reasonable exam expectation is pattern recognition, not verbatim cmdlet memorization.

---

# Bulk Operations Pattern

The standard scripted approach for bulk work the admin center's own bulk-import UI doesn't cover, or where more complex per-row logic is needed:

```powershell
Import-Csv "NewHires.csv" | ForEach-Object {
    New-MgUser -DisplayName $_.DisplayName `
               -UserPrincipalName $_.UPN `
               -UsageLocation $_.UsageLocation `
               -AccountEnabled:$true `
               -PasswordProfile @{ Password = $_.TempPassword; ForceChangePasswordNextSignIn = $true }
}
```

```text
CSV File (one row per user)
        │
        ▼
Import-Csv
        │
        ▼
ForEach-Object { New-MgUser ... }
        │
        ▼
Users Created in Bulk, One Graph Call per Row
```

> [!important] SC-300 tie-in
> Recall from [[Entra Identities]] that **usage location** must be set before license assignment succeeds — a bulk-creation script that omits it will create users that silently can't receive licenses later. This is a realistic, testable failure mode combining a PowerShell scripting detail with a licensing/attribute concept from a different domain.

---

# Least Privilege in Scripting

Just as with Graph API permissions directly, request only the scopes a given script actually needs:

```powershell
# Narrow — script only reads sign-in logs
Connect-MgGraph -Scopes "AuditLog.Read.All"

# Overly broad — avoid unless genuinely required
Connect-MgGraph -Scopes "Directory.ReadWrite.All"
```

---

# Troubleshooting

## Cmdlet Not Found

Check:

1. Whether the relevant submodule (e.g., `Microsoft.Graph.Identity.Governance`) is installed and imported
2. Whether an old, deprecated module (`AzureAD`/`MSOnline`) is still loaded and causing a naming conflict

---

## "Insufficient Privileges" Error Despite Being an Admin

Check:

1. Scopes requested at `Connect-MgGraph` time — the session is limited to what was explicitly consented, not everything the admin's role could theoretically do
2. Whether the operation actually requires an application permission (unattended automation) rather than a delegated one

---

## Bulk Script Partially Succeeds

Check:

1. Per-row error output — most cmdlets return clear errors per failed object rather than failing the whole batch silently
2. Required attributes (usage location, valid UPN format, duplicate detection) on the failing rows specifically

---

# Best Practices

- Always specify `-Scopes` explicitly and narrowly at `Connect-MgGraph` time — don't rely on broad default scope sets
- Migrate any remaining AzureAD/MSOnline scripts to the Microsoft Graph PowerShell SDK proactively, not reactively after deprecation issues arise
- Wrap bulk operations in per-row error handling so partial failures are visible and actionable, not silently swallowed
- Treat scripted bulk operations with the same attribute-completeness discipline as manual creation (usage location, required fields) — automation doesn't bypass Entra ID's own requirements

---

# Related Notes

- [[Microsoft Graph API]]
- [[Entra Identities]]
- [[KQL for Entra ID]]
- [[Privileged Identity Management]]
- [[Entitlement Management]]

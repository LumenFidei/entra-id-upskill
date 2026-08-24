# KQL for Entra ID

## Overview

**Kusto Query Language (KQL)** is used to query Entra ID log data once it's flowing into a **Log Analytics workspace** via diagnostic settings (see [[Monitoring and Logs]]). Where [[Microsoft Graph API]] and [[PowerShell for Entra ID]] are about *managing* directory objects, KQL is purely about *querying and analyzing log data that already exists*.

> [!important] SC-300 tie-in
> The exam's audience profile explicitly names KQL as an expected familiarity, alongside PowerShell. You're not expected to write complex queries from scratch, but you should recognize what a basic query is doing and know KQL applies to Log Analytics specifically — not the Entra admin center's built-in log viewer, which uses its own filter UI instead.

Related:

- [[Monitoring and Logs]]
- [[Identity Protection]]
- [[Privileged Identity Management]]

---

# Where KQL Fits

```text
Entra ID Activity
        │
        ▼
Diagnostic Settings Export (see Monitoring and Logs)
        │
        ▼
Log Analytics Workspace
        │
        ▼
KQL Queries ──── Interactive investigation, custom alerts, workbook data sources
```

> [!important] SC-300 tie-in
> A scenario describing filtering sign-in logs **directly in the Entra admin center** doesn't require KQL at all — that's just the built-in filter UI. KQL only becomes relevant once logs have been routed to Log Analytics. Don't over-apply KQL as the answer to every logging question.

---

# Key Tables for Identity Data

|Table|Contains|
|---|---|
|`SigninLogs`|Interactive user sign-in events|
|`AADNonInteractiveUserSignInLogs`|Non-interactive sign-ins (token refreshes, background auth)|
|`AADServicePrincipalSignInLogs`|Service principal (app/workload identity) sign-ins|
|`AADManagedIdentitySignInLogs`|Managed identity sign-ins specifically|
|`AuditLogs`|Directory changes — role assignments, group membership edits, config changes|
|`AADProvisioningLogs`|Automated provisioning outcomes (cross-tenant sync, SCIM app provisioning)|
|`AADRiskyUsers` / `AADUserRiskEvents`|Identity Protection risk data|

> [!important] SC-300 tie-in
> Notice `SigninLogs` covers only **interactive human** sign-ins — investigating a suspicious **service principal** or **managed identity** requires the separate `AADServicePrincipalSignInLogs` / `AADManagedIdentitySignInLogs` tables. A scenario about a compromised workload identity (see [[App and Azure Workload Identities]] and [[Identity Protection]]'s risky workload identity coverage) points to these tables specifically, not the general sign-in log.

---

# Basic KQL Structure

```text
TableName
| where <condition>
| project <columns to return>
| summarize <aggregation> by <grouping>
| order by <column> [asc|desc]
```

Queries are built as a **pipeline** — each line filters, reshapes, or aggregates the output of the line before it, read top to bottom.

---

# Example Queries (Conceptual)

## Failed Sign-Ins by User, Last 24 Hours

```kql
SigninLogs
| where ResultType != "0"
| where TimeGenerated > ago(24h)
| summarize FailureCount = count() by UserPrincipalName
| where FailureCount > 10
| order by FailureCount desc
```

## Detecting Legacy Authentication Usage

```kql
SigninLogs
| where ClientAppUsed in ("Other clients", "IMAP4", "POP3", "SMTP")
| project TimeGenerated, UserPrincipalName, ClientAppUsed, IPAddress
```

> [!important] SC-300 tie-in
> This connects directly to [[Conditional Access]]'s legacy authentication blocking baseline policy — before blocking legacy auth tenant-wide, a common real-world step is running exactly this kind of query to identify who's still using it, so the policy rollout doesn't unexpectedly break someone's still-active legacy client.

## Recent Privileged Role Assignment Changes

```kql
AuditLogs
| where OperationName == "Add member to role"
| project TimeGenerated, InitiatedBy, TargetResources
```

> [!important] SC-300 tie-in
> This is the kind of query that would supplement — not replace — [[Privileged Identity Management]]'s own audit history view. PIM's built-in audit history has richer eligible/active/approval context specific to PIM; a general `AuditLogs` KQL query is broader but less specialized. Know both exist and serve overlapping-but-distinct investigative purposes.

---

# Time Windowing and Aggregation

|Function/Operator|Purpose|
|---|---|
|`ago(24h)`|Relative time filter (last 24 hours, 7 days, etc.)|
|`bin(TimeGenerated, 1h)`|Buckets timestamps into fixed intervals, for time-series charting|
|`summarize count() by X`|Aggregates and counts rows grouped by a column|
|`summarize ... by bin(TimeGenerated, 1d)`|Common pattern for day-by-day trend queries feeding a workbook chart|

---

# KQL and Workbooks

[[Monitoring and Logs]] covers workbooks as prebuilt visual dashboards — under the hood, most workbook visualizations are themselves powered by KQL queries against these same tables. Building a custom workbook panel generally means writing (or adapting) a KQL query and pointing a chart visualization at its output.

```text
KQL Query
        │
        ▼
Workbook Visualization (chart, table, tile)
        │
        ▼
Reusable, Shareable Dashboard
```

---

# Troubleshooting

## Query Returns No Results Despite Known Activity

Check:

1. Whether diagnostic settings are actually configured to export the relevant log type to this workspace
2. Time window in the query (`ago()` value) relative to when the activity actually occurred
3. Correct table name — a common error is querying `SigninLogs` when the activity was actually a service principal or managed identity sign-in

---

## Investigating a Workload Identity Instead of a Human User

Check:

1. Use `AADServicePrincipalSignInLogs` or `AADManagedIdentitySignInLogs`, not `SigninLogs`
2. Cross-reference with [[Identity Protection]]'s risky workload identity detections for the same principal

---

## Legacy Auth Query Returns Unexpected Results After Blocking Policy Deployed

Check:

1. Whether the Conditional Access policy is actually in "On" state, not still in report-only mode (see [[Conditional Access]])
2. Whether the query's time window predates or postdates the policy's enforcement date

---

# Best Practices

- Run a legacy authentication usage query before deploying a blocking Conditional Access policy, not after users start reporting breakage
- Match the log table to the actual principal type under investigation (human vs. service principal vs. managed identity)
- Convert recurring investigative queries into workbook panels rather than re-running them manually each time
- Treat KQL and PIM's native audit history as complementary tools for privileged access investigations, not substitutes for one another

---

# Related Notes

- [[Monitoring and Logs]]
- [[Identity Protection]]
- [[Privileged Identity Management]]
- [[Conditional Access]]
- [[App and Azure Workload Identities]]

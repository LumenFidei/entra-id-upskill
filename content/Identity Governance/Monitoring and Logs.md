# Monitoring and Logs

## Overview

Identity administration doesn't end at configuration — knowing how to **observe** what's actually happening (sign-ins, administrative changes, provisioning outcomes) and how to **improve posture** over time is its own exam-tested skill set, closing out the Identity Governance domain.

> [!important] SC-300 tie-in
> This note maps to the exam's final bullet group: "Monitor identity activity by using logs, workbooks, and reports." Expect questions on picking the *right log type* and the *right destination* for a given monitoring or compliance requirement.

Related:

- [[Privileged Identity Management]]
- [[Identity Protection]]
- [[Tenant Configuration]]

---

# The Three Core Log Types

|Log|Captures|
|---|---|
|**Sign-in logs**|Every authentication attempt — user, app, Conditional Access policies applied, result, device/location detail|
|**Audit logs**|Directory changes — role assignments, group membership changes, application configuration edits, who made the change|
|**Provisioning logs**|Outcomes of automated provisioning (e.g., cross-tenant sync, SCIM-based app provisioning, HR-driven lifecycle workflows) — created, updated, or failed provisioning events|

```text
Sign-In Logs ──── "Who authenticated, and what happened?"
Audit Logs ──── "What changed in the directory, and who changed it?"
Provisioning Logs ──── "Did an automated provisioning action succeed or fail?"
```

> [!important] SC-300 tie-in
> A classic distractor pattern: a scenario about "a user's group membership changed unexpectedly" sounds like it could be a sign-in log question, but it's actually an **audit log** question — sign-in logs don't record directory changes, only authentication events. Similarly, "an automated SCIM sync stopped creating accounts for new hires" is a **provisioning log** question, not an audit log one.

---

# Reviewing Logs in the Entra Admin Center

All three log types are viewable and filterable directly in the Entra admin center, with export options (CSV, or streaming via diagnostic settings below) for retention beyond the admin center's default log retention window.

```text
Entra Admin Center
        │
        ├── Sign-in Logs (filter by user, app, status, CA result)
        ├── Audit Logs (filter by activity type, initiated by, target)
        └── Provisioning Logs (filter by provisioning job, status)
```

> [!important] SC-300 tie-in
> Default log retention in the Entra admin center is limited (commonly a matter of days to a few weeks depending on license and log type) — a scenario requiring **long-term retention** or **compliance-driven audit history** requires configuring diagnostic settings to export logs elsewhere, not just relying on the admin center's built-in view.

---

# Diagnostic Settings

Configures where logs are **streamed** for longer retention, deeper analysis, or SIEM integration.

|Destination|Best Fit|
|---|---|
|**Log Analytics workspace**|Interactive querying via KQL, workbooks, alerting|
|**Storage account**|Long-term, low-cost archival for compliance retention requirements|
|**Azure Event Hubs**|Streaming to external SIEM tools (e.g., Splunk, or non-Microsoft platforms) in near real time|

```text
Entra ID Logs
        │
        ├── Log Analytics Workspace ──── KQL queries, workbooks, alerts
        ├── Storage Account ──── Long-term archive (compliance)
        └── Event Hub ──── External SIEM streaming
```

> [!important] SC-300 tie-in
> A scenario emphasizing **"query and investigate interactively"** points to Log Analytics. A scenario emphasizing **"retain for 7 years for compliance"** points to a storage account. A scenario emphasizing **"feed into our existing third-party SIEM"** points to Event Hubs. These three destinations aren't interchangeable — picking the wrong one is a common distractor.

---

# Monitoring with KQL in Log Analytics

Once logs are flowing into a Log Analytics workspace, **Kusto Query Language (KQL)** is used to search, filter, and aggregate them for investigation or custom alerting.

```text
Example Shape (conceptual, not literal syntax)

SigninLogs
| where ResultType != "0"
| where TimeGenerated > ago(24h)
| summarize FailureCount = count() by UserPrincipalName
| where FailureCount > 10
```

> [!important] SC-300 tie-in
> The exam's audience profile explicitly lists **KQL familiarity** as an expected skill — you're not expected to be a KQL expert, but you should recognize what a basic query is doing conceptually (filtering, summarizing, time-windowing) and know that KQL is the query language for Log Analytics specifically, not for the Entra admin center's built-in log viewer.

---

# Workbooks

Pre-built (and customizable) visual dashboards in the Entra admin center and Azure Monitor, built on top of the same underlying log data — used to analyze trends (sign-in failures over time, Conditional Access impact, risky sign-in trends) without writing KQL from scratch each time.

```text
Raw Logs (Sign-In, Audit, Provisioning)
        │
        ▼
Workbooks
        │
        ▼
Visual Trend Analysis, Reusable Dashboards
```

> [!important] SC-300 tie-in
> Workbooks are the answer whenever a scenario emphasizes a **recurring, visual, shareable report** — for example, a monthly Conditional Access impact summary for leadership — versus a one-off investigative KQL query, which is more appropriate for ad hoc troubleshooting.

---

# Identity Secure Score

A percentage-based score reflecting how well the tenant's current configuration aligns with Microsoft's security recommendations, alongside a prioritized list of specific improvement actions (e.g., "enable MFA for all admins," "designate more than one Global Administrator," "turn on Identity Protection sign-in risk policy").

```text
Tenant Configuration
        │
        ▼
Compared Against Microsoft's Recommended Baseline
        │
        ▼
Identity Secure Score (percentage)
        │
        ▼
Prioritized, Actionable Improvement Recommendations
```

> [!important] SC-300 tie-in
> Identity Secure Score is explicitly named in the exam bullet list ("Monitor and improve the security posture by using Identity Secure Score") — know it as a specific, named feature with actionable recommendations, not just a generic dashboard metric. A scenario asking "how can we get a prioritized list of security improvements without hiring a consultant" is describing this feature directly.

---

# Troubleshooting

## Investigating an Unexpected Directory Change

Check:

1. Audit logs, filtered by the target object and time window
2. Whether the change originated from a person (interactive) or an automated process (check the "initiated by" field, including service principals)

---

## Automated Account Creation Not Happening for New Hires

Check:

1. Provisioning logs for the relevant job (cross-tenant sync, SCIM app provisioning, or lifecycle workflow) — look for failure reasons, not just absence of success entries
2. Source attribute completeness (a common provisioning failure is a missing required attribute on the source record)

---

## Compliance Team Needs Multi-Year Log Retention

Check:

1. Whether diagnostic settings are configured to export to a storage account (not just relying on default admin-center retention)
2. Retention policy configured on the storage account itself, separate from the Entra-side export configuration

---

# Best Practices

- Configure diagnostic settings early — don't wait until an incident to discover default retention wasn't sufficient
- Route logs to the destination that matches the actual need: Log Analytics for investigation, storage accounts for compliance archival, Event Hubs for SIEM integration
- Build workbooks for any report that will be requested more than once, rather than re-running the same KQL query manually each time
- Review Identity Secure Score recommendations on a regular cadence, not just once at initial tenant setup

---

# Related Notes

- [[Privileged Identity Management]]
- [[Identity Protection]]
- [[Tenant Configuration]]
- [[Access Reviews]]

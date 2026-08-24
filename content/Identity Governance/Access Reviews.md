# Access Reviews

## Overview

**Access Reviews** provide a recurring, structured process for verifying that existing access — to groups, applications, roles, or access packages — is still needed, rather than assuming access remains appropriate indefinitely once granted.

> [!important] SC-300 tie-in
> Access Reviews require a **P2** license, same as Entitlement Management and PIM (see [[Entra ID Editions and Licensing]]). Access Reviews are the *periodic verification* mechanism — distinct from [[Entitlement Management]], which handles the *initial request and time-bound grant*. The exam tests both as complementary, not interchangeable.

Related:

- [[Entitlement Management]]
- [[Privileged Identity Management]]
- [[Entra ID Editions and Licensing]]

---

# What Access Reviews Can Review

```text
Access Reviews Can Target
        │
        ├── Group Memberships (security groups, M365 groups)
        ├── Application Assignments (Enterprise Applications)
        ├── Access Package Assignments (Entitlement Management)
        └── Entra/Azure Role Assignments (via PIM)
```

---

# Planning an Access Review

Key decisions before creating a review:

|Decision|Options|
|---|---|
|Reviewer|Self-review (users attest to their own access), manager, resource owner, or a specific selected group of reviewers|
|Frequency|One-time, or recurring (weekly, monthly, quarterly, annually)|
|Scope|All members, or only guest users specifically|
|Action on no response|Approve, deny, or take the recommended (AI-assisted) action automatically|

```text
Plan Access Review
        │
        ├── Who is being reviewed?
        ├── Who reviews them?
        ├── How often does this recur?
        └── What happens if nobody responds in time?
```

> [!important] SC-300 tie-in
> **Self-review** (having users attest to their own continued need for access) is a legitimate, commonly recommended configuration — don't assume every access review must be manager- or owner-driven. A scenario emphasizing scale and minimizing reviewer burden for lower-risk resources often points toward self-review.

---

# Creating and Configuring Access Reviews

```text
Define Scope (which resource, which population)
        │
        ▼
Assign Reviewer(s)
        │
        ▼
Set Recurrence and Duration
        │
        ▼
Configure Auto-Apply Results (optional)
        │
        ▼
Configure Fallback Action for No Response
        │
        ▼
Review Runs and Collects Decisions
```

## Auto-Apply Results

Access reviews can be configured to **automatically apply** the outcome (e.g., automatically remove denied users from a group) rather than requiring a separate manual step after the review period closes.

> [!important] SC-300 tie-in
> Without auto-apply enabled, a completed access review's decisions sit inert until an administrator manually applies them — a scenario describing "the review finished but access wasn't actually removed" is very often this missing configuration, not a review failure.

---

# Monitoring Access Review Activity

Administrators can track review progress in real time — how many reviewers have responded, how many decisions are pending, and what the interim results look like before the review period closes.

---

# Manually Responding to Access Review Activity

When automated recommendations or reviewer inaction leave decisions pending, administrators can manually approve, deny, or override individual review decisions — including acting on an access review's **AI-assisted recommendation**, which suggests a decision based on factors like the user's recent sign-in activity to that resource.

```text
Reviewer Doesn't Respond
        │
        ▼
Recommendation Engine Suggests a Decision
(e.g., "Deny — no sign-in activity in 90 days")
        │
        ▼
Admin Can Accept Recommendation or Override Manually
```

> [!important] SC-300 tie-in
> The recommendation engine is based on observed usage/sign-in patterns, not a guess — know that this is a real, named feature ("Recommendations" in access reviews) and not just a generic reminder nudge. A scenario asking "how can reviewers make faster, more informed decisions at scale" is pointing at this feature.

---

# Access Reviews for Privileged Roles

Access reviews are also the standard mechanism for periodically re-certifying **privileged role assignments** managed through PIM (see [[Privileged Identity Management]]) — ensuring that eligible or active role holders still genuinely need that access, not just group/app membership.

```text
PIM Role Assignment (e.g., Global Administrator)
        │
        ▼
Recurring Access Review Targets This Role
        │
        ▼
Assigned Admins Periodically Re-Justify Their Access
        │
        ▼
Unjustified/Unused Assignments Removed
```

---

# Troubleshooting

## Review Completed But Access Was Not Removed

Check:

1. Auto-apply setting on the review — likely disabled, requiring a manual apply step
2. Whether the review actually reached a "completed" state versus still being in progress

---

## Reviewers Not Responding

Check:

1. Whether reminder notifications are configured and firing correctly
2. Whether the fallback "no response" action is set appropriately for the resource's risk level (approve vs. deny by default has very different risk implications)

---

## Guest Users Not Being Captured by a Review Intended for Them

Check:

1. Review scope setting — confirm it's targeting "guest users only" rather than "all users," if that was the intent
2. Whether the guest's access actually originates from the resource being reviewed, versus a different, unreviewed path

---

# Best Practices

- Use self-review for lower-risk, high-volume resources to reduce reviewer burden without sacrificing governance entirely
- Always enable auto-apply for recurring reviews, to avoid decisions sitting unapplied after each cycle
- Set a deny-by-default fallback action for higher-risk resources (privileged roles, sensitive apps) rather than approve-by-default
- Pair access reviews with entitlement management access packages for a complete request-then-verify governance loop, rather than using either in isolation

---

# Related Notes

- [[Entitlement Management]]
- [[Privileged Identity Management]]
- [[Entra ID Editions and Licensing]]
- [[Monitoring and Logs]]

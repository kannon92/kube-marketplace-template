---
description: Example of triaging GitHub notifications over the past week
---

## Name

triage-github-notifications:example

## Synopsis

```
/triage-github-notifications
```

## Description

Triage your GitHub notifications. The command will prompt for your GitHub username and the number of days to look back, then fetch and categorize notifications by reason (review requested, assigned, mentioned, etc.), presenting actionable items in a prioritized table.

## Implementation

1. Invoke the command with no arguments.
2. When prompted, enter your GitHub username and the desired lookback window (e.g., 7 days).
3. The command fetches notifications via `gh api /notifications` and groups them by reason.

## Return Value

A prioritized table of GitHub notifications that require action, with direct links to each item.

---
description: Example of triaging a failing Prow job by URL
---

## Name

triage-prow-job:example

## Synopsis

```
/triage-prow-job https://prow.k8s.io/view/gs/kubernetes-ci-logs/logs/ci-kubernetes-e2e-gci-gce/1234567890
```

## Description

Triage a failing Prow job by passing its URL. The command fetches the build artifacts from GCS, parses test results and logs, identifies the root cause, and searches GitHub for existing issues.

## Implementation

1. Pass the full Prow job URL as the argument.
2. The command extracts the GCS path from the URL and fetches `finished.json`, `build-log.txt`, and JUnit XML artifacts.
3. It identifies failing tests and analyzes logs for error messages.
4. It searches GitHub issues in the relevant repository for existing reports.

## Return Value

A structured triage summary with failing tests, root cause analysis, and links to any related GitHub issues or PRs.

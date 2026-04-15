---
name: triage-prow-job
description: Triage a failing Prow job from its URL
user-invocable: true
---

## Name

triage-prow-job:triage-prow-job

## Synopsis

Given a Prow job URL, analyze the failure and search for existing issues.

## Description

Triage a failing Prow job by fetching its artifacts, analyzing test results, identifying root causes, and searching for existing GitHub issues.

The argument is the Prow URL: $ARGUMENTS

## Implementation

## Step 1: Parse the URL

Extract the GCS bucket path from the Prow URL. Prow URLs follow this pattern:
`https://prow.k8s.io/view/gs/<bucket>/<path>/<job-name>/<build-id>`

The GCS base path is: `gs://<bucket>/<path>/<job-name>/<build-id>`
The HTTPS base for fetching is: `https://storage.googleapis.com/<bucket>/<path>/<job-name>/<build-id>`

## Step 2: Fetch job metadata

Fetch these files (use WebFetch for HTTPS URLs, or gsutil if available):
- `finished.json` - job result, version, timestamp
- `build-log.txt` - the build log (may be large/truncated; focus on errors and the end of the log)

Summarize: job name, result, version, when it ran.

## Step 3: Get test results

Use `gsutil ls` to list the artifacts directory:
`gsutil ls "<gcs-base>/artifacts/"`

Then fetch all `junit*.xml` files and extract:
- Total tests, passed, failed, skipped counts
- For each failure: the test name, the failure message, and the file/line where it failed

## Step 4: Analyze the failure

For each failing test:
- Read the relevant test source code in the local repo to understand what the test expects
- Determine whether this is a test bug, a product bug, or an infrastructure issue
- Identify the root cause or most likely explanation

## Step 5: Search for existing issues

Search kubernetes/kubernetes for existing issues using multiple queries:
- The exact test name
- Key error messages or symptoms
- The job name
- Related feature gates or components

Check both open and closed issues. For relevant closed issues, note whether they were truly fixed or just closed without resolution.

## Step 6: Report findings

Present a structured summary:
1. **Job**: name, build ID, result, version
2. **Failing test(s)**: name, failure details, root cause analysis
3. **Existing issues**: any open or recently closed issues that match
4. **Recommendation**: whether to file a new issue, reopen an existing one, or if the failure is already tracked

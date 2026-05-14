# triage-prow-job

Triage a failing Prow job by fetching its artifacts, analyzing test results, and identifying root causes. Given a Prow job URL, the command downloads build logs and JUnit XML artifacts from GCS, summarizes failures, and searches GitHub for existing issues.

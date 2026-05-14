# triage-prow-job

Triage a failing Prow job by fetching its artifacts, analyzing test results, identifying root causes, and searching for existing GitHub issues. Given a Prow job URL, the command downloads build logs and JUnit XML artifacts from GCS, summarizes failures, and searches GitHub for existing issues.

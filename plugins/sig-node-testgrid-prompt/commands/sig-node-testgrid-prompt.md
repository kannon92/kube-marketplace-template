---
name: sig-node-testgrid-prompt
description: Regenerate the sig-node TestGrid triage report
user-invocable: true
---

## Name

sig-node-testgrid-prompt:sig-node-testgrid-prompt

## Synopsis

Regenerate the sig-node TestGrid triage report from scratch.

## Description

Use this in a fresh Claude Code session to regenerate the sig-node triage report.

Goal:
- update the local report at `~/notes/sig-node-testgrid-triage.md`
- keep this prompt file at `~/notes/sig-node-testgrid-triage-prompt.md`

## Implementation

Copy-paste the block below:

````
I need a thorough, evidence-based triage of the Kubernetes sig-node TestGrid dashboard.

Primary dashboard: https://testgrid.k8s.io/sig-node/

You are expected to regenerate the full markdown report from scratch, not just summarize current failures.

## Goals

1. Enumerate every FAILING and FLAKY tab across all sig-node sub-dashboards.
2. Build a dashboard-level summary table with pass/flaky/fail counts for each sub-dashboard.
3. For each failing or flaky job, determine:
   - current status
   - failing test names
   - exact error messages / symptoms
   - likely root cause
   - whether it is a regression, pre-existing flake, or infra/capacity issue
   - first bad build / regression boundary when possible
4. Identify patterns across jobs:
   - same test failing in multiple jobs
   - same root cause cascading into many failures
   - jobs that are red only because of infrastructure limitations
5. Rank fixes by ROI:
   - what single fix would recover the most jobs?
   - what fixes are already merged and simply propagating?
   - what needs a new PR vs an existing PR review/rebase?
6. Check for recovery evidence in the newest builds:
   - if a fix merged recently, inspect post-merge builds and say whether recovery is confirmed, partial, or not yet visible
7. Regenerate the full markdown report, and keep the reusable prompt in sync if you improve it.

## Repos on disk

Can you prompt user for the following repos (kubernetes, test-infra) and then use those from now on

## Files on disk

- report: `~/notes/sig-node-testgrid-triage.md`
- prompt: `~/notes/sig-node-testgrid-triage-prompt.md`

At the end of the triage:

1. ensure `~/notes/sig-node-testgrid-triage.md` exists and is updated
2. ensure `~/notes/sig-node-testgrid-triage-prompt.md` exists and is updated if needed
3. report the local file paths in the final answer

## Dashboards to cover

At minimum, inspect every tab under the sig-node root and produce consolidated coverage for:

- sig-node-release-blocking
- sig-node-kubelet
- sig-node-containerd
- sig-node-cri-o
- sig-node-dynamic-resource-allocation
- sig-node-presubmits
- sig-node-ec2
- sig-node-gpu
- any additional sig-node sub-dashboard currently linked from the root page

If tab counts changed because jobs were renamed, removed, or consolidated, call that out explicitly.

## Investigation requirements

- If a `k8s-ci-triage` skill is available, use it for structured investigation; otherwise proceed manually.
- Launch multiple agents in parallel to cover different failure areas simultaneously.
- Use the TestGrid table JSON endpoint as the default first-pass analysis path for each failing or flaky tab:
  - `https://testgrid.k8s.io/<dashboard>/table?tab=<tab>&dashboard=<dashboard>&exclude-non-failed-tests=10`
  - fetch it with `curl -sL`
  - use it to extract exact row names, recent cell values, timestamps, build IDs, and per-row failure messages before reading screenshots or manually scanning the UI
- Treat `exclude-non-failed-tests=10` as the default diagnosis window unless you explicitly need a longer history span.
- Use `https://storage.googleapis.com/k8s-metrics/flakes-latest.json` as a secondary signal after the TestGrid JSON pass:
  - use it to flag jobs and exact tests with recent historical instability
  - use it to prioritize follow-up when it overlaps with current TestGrid rows
  - do not treat it as the source of truth for current state when it disagrees with live TestGrid data
- Use actual CI artifacts before proposing or endorsing a fix.
- Prefer `gh` for GitHub data, `curl` for web fetches, and `gsutil` to download CI logs/artifacts from GCS.
- Do not rely only on TestGrid row text; inspect artifacts.

## Minimum evidence for each failing or flaky job

For each investigated tab, first collect the current row matrix from the TestGrid JSON endpoint:

- `https://testgrid.k8s.io/<dashboard>/table?tab=<tab>&dashboard=<dashboard>&exclude-non-failed-tests=10`

From that JSON, extract at least:

- exact row names from `tests[].name`
- recent row outcomes from `tests[].short_texts`
- row-level failure messages from `tests[].messages`
- recent build IDs from `changelists`
- recent timestamps from `timestamps`

Use those row outcomes to classify each row before inspecting artifacts:

- current failing:
  - latest visible cell is `F`, or a fraction like `0/2` or `1/2`
- recent flaky / not currently failing:
  - latest visible cell is clean, but some recent visible cell is `F` or an incomplete fraction like `1/2`
- clean / filtered away:
  - no fail-like cells in the visible window

Treat `F` and any `x/y` where `x < y` as fail-like. Treat `S` as skipped, not failing.
Use this row classification to identify failure families, but do not treat it by itself as proof that the lane is still red at head. Filtered TestGrid tables can still surface historical failing rows on stale-red tabs whose newest build already succeeded.
Use this JSON pass to identify:

- which exact tests are currently red
- which tests are only recent flakes
- which rows overlap across multiple tabs
- whether a lane is dominated by one narrow deterministic row family or a broad unstable cluster

Then fetch the flake summary dataset:

- `https://storage.googleapis.com/k8s-metrics/flakes-latest.json`

From that dataset, extract for each relevant job when present:

- job-level `consistency`
- job-level `flakes`
- per-test entries from `test_flakes`

Use `flakes-latest.json` only as a secondary signal:

- overlap with current TestGrid rows:
  - strong evidence that the current failure belongs to a recurring instability family
- present in flakes data but not currently red in TestGrid:
  - put on the watch list as a latent or recently cooled flake family
- currently red in TestGrid but absent from flakes data:
  - do not dismiss it; this often means the failure is newer, more deterministic, or simply not represented in the flake summary window

If a presubmit or canary lane is missing under its raw Prow job name, also check the `pr:`-prefixed variant in `flakes-latest.json`.

For each red or flaky job, inspect enough recent builds to answer:

- What is the latest failing build?
- Is there a recent passing build? If yes, when and at what SHA?
- Is the failure consistent, intermittent, or newly recovered?
- Which test(s) fail first, and are later failures cascade effects?

Before you call a lane actively red at head, independently verify the newest build result with `finished.json`, current Prow history, or current GitHub checks. Do not equate `tests[].short_texts[0]` with head failure on its own.

For each investigated build, collect evidence from some combination of:

- Prow history:
  - `https://prow.k8s.io/job-history/gs/kubernetes-ci-logs/logs/<job-name>`
- Build metadata:
  - `gs://kubernetes-ci-logs/logs/<job-name>/<build-id>/started.json`
  - `gs://kubernetes-ci-logs/logs/<job-name>/<build-id>/finished.json`
- Artifacts directory:
  - `gs://kubernetes-ci-logs/logs/<job-name>/<build-id>/artifacts/`
- Typical useful files:
  - `build-log.txt`
  - `junit*.xml`
  - `podinfo.json`
  - kubelet/containerd/crio/journal logs when present

Use the TestGrid JSON output to choose which rows and builds to inspect, then confirm the leading row families against artifacts.
Use `gsutil -m cp -r` to download the artifacts you need into `/tmp/` and inspect them locally.
If a build has no JUnit or no obvious test summary, say so and use `build-log.txt` and metadata instead.
Before declaring a periodic lane inaccessible, first check the Prow job-history page under `gs/kubernetes-ci-logs/logs/<job-name>` and confirm the actual bucket/path in current use; some lanes that older notes describe as private are currently readable there.
If a bucket or exact artifact path is still unreadable from the environment after that check, say so explicitly and fall back to current Prow/TestGrid state, job config, accessible sibling lanes, and GitHub issue/PR evidence. Do not overstate certainty in that case.
For presubmit jobs shown under `pr-logs/directory`, recover the concrete GCS path from the Prow job-history page's `SpyglassLink` first; it will usually be of the form `gs://kubernetes-ci-logs/pr-logs/pull/<pr-number>/<job-name>/<build-id>/`.

## How to investigate

1. Start from the current sig-node TestGrid root page and enumerate all red/flaky tabs.
2. For each red/flaky tab, fetch the TestGrid table JSON with `exclude-non-failed-tests=10` and extract exact current-failing and recent-flaky rows.
3. Fetch `flakes-latest.json` and map the relevant jobs and rows into the same investigation set.
4. Group jobs by likely failure family as early as possible, using row-name overlap across tabs before diving into logs.
   - compare current TestGrid rows with historical `flakes-latest.json` rows
   - use overlaps to identify recurring families
   - use mismatches to separate new deterministic failures from old background flakes
5. For each family:
   - identify the exact current-failing rows from TestGrid JSON
   - separate them from recent-flaky-only rows
   - compare row overlap across related tabs to identify shared families vs isolated lanes
   - compare those rows against `flakes-latest.json` to see which tests have recent historical instability
   - download representative logs for the leading rows and builds
   - identify the first meaningful error, not just the last cascade failure
   - inspect source code in the kubernetes checkout
   - inspect job definitions in `test-infra/config/jobs/kubernetes/sig-node/`
6. For regressions:
   - find the regression window from build history, SHAs, and the TestGrid row matrix
   - inspect recent PRs/commits that plausibly introduced the behavior
7. For fixes already in flight:
   - search GitHub for issues and PRs with `gh`
   - inspect the relevant PR diffs and current status
   - note whether the proposed fix is sufficient, partial, or needs follow-up
8. For fixes on local branches or in local checkouts:
   - inspect local code and current branch state when relevant
   - note branch names, commit SHAs, whether fixes were pushed/opened, and whether local branches are clean or diverged from origin
9. Distinguish root causes from symptoms:
   - if one kubelet panic causes 20 downstream test failures, count that as one root cause with a cascade, not 20 unrelated bugs

## GitHub and code search

- Search issues with `gh search issues --repo kubernetes/kubernetes '<term>'`
- Search PRs with `gh search prs --repo kubernetes/kubernetes '<term>'`
- Use `gh pr view`, `gh pr checks`, and `gh issue view` for current state
- Use ripgrep in the local repos to find tests, feature gates, job names, and config

## Prior report

There is a prior report at:
`~/notes/sig-node-testgrid-triage.md`

Read it first, but do not trust it blindly.

- Re-verify every major claim against current CI state.
- Mark items as recovered, partially recovered, still failing, or obsolete.
- Preserve useful structure and context from the prior report.
- Refresh counts, build IDs, SHAs, open PR statuses, and ranking.
- If a previously proposed fix turned out to be incomplete, say so clearly.
- If the prior report missed a new failure family, add it.

If no prior report exists, create the report from scratch using the same section structure.

## Output requirements

Write the updated report to:
`~/notes/sig-node-testgrid-triage.md`

Keep the reusable prompt in:
`~/notes/sig-node-testgrid-triage-prompt.md`

The report must include these sections:

- `# Sig-Node TestGrid Triage Report -- YYYY-MM-DD`
- `## Executive Summary`
- `## Ranked Fixes by Impact`
- `## All Failing Jobs (Consolidated)`
- `## Recovery Timeline`
- `## Top 20 Actions by ROI`
- `## Key GitHub Issues`
- `## Key PRs`
- `## Work Done During This Triage`
- `## Prompt`

Within the report:

- Under `## Executive Summary`, include these subsections:
  - `### Recovery Status`
  - `### Dashboard Overview`
- Include a dashboard overview table with pass/flaky/fail counts.
- Include a recovery status table with before/after evidence where applicable.
- In Ranked Fixes, use numbered sections with enough detail to stand alone:
  - impact
  - affected jobs
  - failing tests
  - symptom
  - root cause
  - issue/PR links
  - current fix status
  - recovery evidence when available
- In All Failing Jobs, consolidate by dashboard and map each job to a root cause.
  - ensure every currently failing or flaky tab from the dashboard review is either listed there or explicitly explained as a duplicate/alias/recovered item
- In Recovery Timeline, include date/time, event, and current status.
- In Top 20 Actions, group by:
  - Completed
  - Active PRs / in progress
  - Needs new PR / follow-up
- In Key GitHub Issues and Key PRs, use markdown tables.
- In Work Done, list concrete work performed during the session.
  - if the session created or updated fixes, include local branch names, final commit SHAs, pushed remotes, and PR numbers
- In `## Prompt`, point readers to `~/notes/sig-node-testgrid-triage-prompt.md`
- Explicitly account for all dashboard tabs:
  - every red or flaky tab must either appear in the consolidated section or be marked as a duplicate, alias, stale-red, or externally owned lane
- Cross-check that dashboard totals and the consolidated inventory are internally consistent.

## Link formatting

- All PR references must be markdown links using `/pull/`
  - example: `[k/k #12345](https://github.com/kubernetes/kubernetes/pull/12345)`
- All issue references must be markdown links using `/issues/`
  - example: `[k/k #12345](https://github.com/kubernetes/kubernetes/issues/12345)`
- Use repository-qualified links when not in kubernetes/kubernetes.

## Quality bar

- Be explicit about uncertainty.
- Do not claim a fix works unless you checked post-fix builds.
- Do not propose code changes before inspecting real logs.
- Do not rely on screenshots or manual browser scanning when the exact row and cell data is available from the TestGrid JSON endpoint.
- Do not treat `flakes-latest.json` as current truth. It is a secondary historical signal that should refine prioritization, not override live TestGrid evidence.
- Do not call a lane head-red from filtered row data alone; verify the newest build result separately because stale-red dashboards can keep historical fail rows visible after head is green.
- Call out when a job is red because of infra capacity rather than test logic.
- Prefer exact build IDs, SHAs, job names, failing test names, and file paths over vague summaries.
- Cross-check that dashboard counts and consolidated job lists are internally consistent.
- If a dashboard is red only because of stale history while the newest runs are green, say that clearly.
- Treat duplicate tabs across dashboards as one underlying problem, but still enumerate them.
- Use multiple parallel agents where it meaningfully reduces investigation time.
````

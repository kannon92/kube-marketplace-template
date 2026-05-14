# Available Plugins

This document lists all available Claude Code plugins and their commands in the ai-helpers repository.

- [Commit From Changes](#commit-from-changes-plugin)
- [Create Pr From Changes](#create-pr-from-changes-plugin)
- [Kube Api Review](#kube-api-review-plugin)
- [Sig Node Testgrid Prompt](#sig-node-testgrid-prompt-plugin)
- [Triage Github Notifications](#triage-github-notifications-plugin)
- [Triage Issue](#triage-issue-plugin)
- [Triage Prow Job](#triage-prow-job-plugin)

### Commit From Changes Plugin

Stage and commit changes with a Kubernetes-style commit message

**Commands:**
- **`/commit-from-changes:commit-from-changes`** - Stage and commit changes with a Kubernetes-style commit message

See [plugins/commit-from-changes/README.md](plugins/commit-from-changes/README.md) for detailed documentation.

### Create Pr From Changes Plugin

Create a GitHub pull request from the current branch

**Commands:**
- **`/create-pr-from-changes:create-pr-from-changes`** - Create a GitHub pull request from the current branch
- **`/create-pr-from-changes:example`** - Example of creating a pull request from the current branch

See [plugins/create-pr-from-changes/README.md](plugins/create-pr-from-changes/README.md) for detailed documentation.

### Kube Api Review Plugin

Review Kubernetes API types against API conventions

**Commands:**
- **`/kube-api-review:kube-api-review`** - Review Kubernetes API types against API conventions

See [plugins/kube-api-review/README.md](plugins/kube-api-review/README.md) for detailed documentation.

### Sig Node Testgrid Prompt Plugin

Triage the sig-node TestGrid dashboard and generate a comprehensive failure report

**Commands:**
- **`/sig-node-testgrid-prompt:sig-node-testgrid-prompt`** - Regenerate the sig-node TestGrid triage report

See [plugins/sig-node-testgrid-prompt/README.md](plugins/sig-node-testgrid-prompt/README.md) for detailed documentation.

### Triage Github Notifications Plugin

Fetch and triage GitHub notifications, highlighting items that require action

**Commands:**
- **`/triage-github-notifications:example`** - Example of triaging GitHub notifications over the past week
- **`/triage-github-notifications:triage-github-notifications`** - Fetch and triage GitHub notifications, highlighting items that require action

See [plugins/triage-github-notifications/README.md](plugins/triage-github-notifications/README.md) for detailed documentation.

### Triage Issue Plugin

Triage a GitHub issue by investigating the codebase, identifying the root cause, and proposing a fix

**Commands:**
- **`/triage-issue:example`** - Example usage of the triage-issue command
- **`/triage-issue:triage-issue`** - Triage a GitHub issue by investigating the codebase, identifying the root cause, and proposing a fix

See [plugins/triage-issue/README.md](plugins/triage-issue/README.md) for detailed documentation.

### Triage Prow Job Plugin

Triage a failing Prow job by fetching its artifacts, analyzing test results, identifying root causes, and searching for existing GitHub issues

**Commands:**
- **`/triage-prow-job:example`** - Example of triaging a failing Prow job by URL
- **`/triage-prow-job:triage-prow-job`** - Triage a failing Prow job from its URL

See [plugins/triage-prow-job/README.md](plugins/triage-prow-job/README.md) for detailed documentation.

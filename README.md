# kubernetes-maintainer-skills

Claude Code Plugins by kannon92

## Installation

Add the marketplace to Claude Code:

```
/plugin marketplace add kannon92/kubernetes-maintainer-skills
```

Install a specific plugin:

```
/plugin install <plugin-name>@kubernetes-maintainer-skills
```

<!-- BEGIN PLUGINS -->
## Plugins

- **commit-from-changes**: Stage and commit changes with a Kubernetes-style commit message
- **create-pr-from-changes**: Create a GitHub pull request from the current branch
- **kube-api-review**: Review Kubernetes API types against API conventions
- **sig-node-testgrid-prompt**: Triage the sig-node TestGrid dashboard and generate a comprehensive failure report
- **triage-github-notifications**: Fetch and triage GitHub notifications, highlighting items that require action
- **triage-issue**: Triage a GitHub issue by investigating the codebase, identifying the root cause, and proposing a fix
- **triage-prow-job**: Triage a failing Prow job by fetching its artifacts, analyzing test results, identifying root causes, and searching for existing GitHub issues
<!-- END PLUGINS -->

## Development

Run the linter to validate plugin structure:

```bash
make lint
```

Update plugin documentation and website:

```bash
make update
```

## Documentation

Visit the [documentation site](https://kannon92.github.io/kubernetes-maintainer-skills) for more information.

## License

MIT

---
description: Example of creating a pull request from the current branch
---

## Name

create-pr-from-changes:example

## Synopsis

```
/create-pr-from-changes
```

## Description

Create a pull request for changes on the current branch. The command inspects the commits on the branch, looks for any pull request templates in the repository, and generates a descriptive title and body before opening the PR with `gh pr create`.

## Implementation

1. The command is invoked with no arguments on a branch that has commits not yet in a pull request.
2. It searches for PR templates at `.github/pull_request_template.md` and similar locations.
3. It generates a PR title and body from the commit messages and staged diff.
4. It runs `gh pr create` with the generated title and body.

## Return Value

A link to the newly created GitHub pull request.

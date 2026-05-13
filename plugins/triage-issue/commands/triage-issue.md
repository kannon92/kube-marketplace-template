---
name: triage-issue
description: Triage a GitHub issue by investigating the codebase, identifying the root cause, and proposing a fix
user-invocable: true
---

## Name

triage-issue:triage-issue

## Synopsis

Given a GitHub issue number or URL, analyze the issue, investigate the codebase, and post a structured analysis comment.

## Description

Triage a GitHub issue by fetching its details, searching the codebase for the root cause, assessing the scope and impact, and proposing a fix. Posts the analysis as a structured comment on the issue.

The argument is the issue number or URL: $ARGUMENTS

## Implementation

### Step 1: Fetch the issue

Parse the issue number or URL from `$ARGUMENTS`. If only a number is provided, use the current repo.

```bash
gh issue view <NUMBER> --repo <REPO> --json number,title,body,labels,author,createdAt,url
```

### Step 2: Understand the problem

Read the issue description carefully and identify:
- The reported symptom (what is wrong)
- The expected behavior (what should happen)
- Environment details and reproduction steps

### Step 3: Investigate the codebase

- Search for relevant code using keywords from the issue (metric names, function names, error messages)
- Trace the full code path from input to the reported symptom
- Identify the exact location where the bug occurs (file, line number, function)
- Look for similar patterns in the codebase that handle the same scenario without the bug
- Check git history for recent changes that may have introduced the bug

### Step 4: Identify scope and impact

- Determine which components, metrics, or features are affected
- Determine which components are NOT affected and why (e.g., they use a correct code path)
- Assess whether the bug affects correctness, performance, or reporting only

### Step 5: Propose a fix

- Describe the specific code changes needed
- Reference existing patterns or helper functions that should be used
- List all files that need changes (including test files)
- Assess the risk level of the fix

### Step 6: Post the analysis as a comment on the issue

Use `gh issue comment` to post a structured comment with the heading `## AI Analysis` and the following sections:
- **Root Cause:** What is wrong and where in the code (include file paths and line numbers)
- **Code Path:** The buggy code path, with references to how similar code handles it without the bug
- **Affected Components:** List what is broken
- **Fix:** The changes needed, files involved, and risk level

Use code blocks for code paths and references. Be specific with file paths and line numbers.

## Return Value

A structured comment posted to the GitHub issue containing root cause analysis, affected components, and a proposed fix.

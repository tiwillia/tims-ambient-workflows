# /ambient-fork-review - Review a Pull Request

## User Input

```text
$ARGUMENTS
```

## Purpose

Perform a comprehensive code review of a pull request for the Ambient Code Platform and post the findings as a comment on the PR.

## Prerequisites

- `gh` CLI is authenticated and available
- A PR URL or number is provided (e.g., `https://github.com/tiwillia/platform/pull/123` or just `123`)

## Process

1. Parse the PR reference from the user input. Extract the owner, repo, and PR number. If only a number is given, default to `tiwillia/platform`.
2. Read the ambient-fork-pr-review skill at `.claude/skills/ambient-fork-pr-review/SKILL.md`
3. Follow the skill's review process:
   - Fetch PR diff and metadata using `gh`
   - Categorize changed files by component
   - Apply all applicable review checks
   - Classify findings by severity
   - Produce the structured review report
4. Post the review as a comment on the PR using `gh pr comment`
5. Save a local copy to `artifacts/pr-review/pr-<number>-review.md`
6. Summarize the results to the user

## Output

- PR comment posted on the pull request
- `artifacts/pr-review/pr-<number>-review.md`

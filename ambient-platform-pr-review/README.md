# Ambient Platform PR Review Workflow

An ACP workflow that reviews pull requests for the Ambient Code Platform against its documented best practices, security standards, and architectural conventions.

## Usage

1. Select this workflow in ACP (or load via Custom Workflow)
2. Provide a PR URL or number: `/review https://github.com/tiwillia/platform/pull/123`
3. The workflow fetches the PR diff, analyzes it against platform standards, and posts a structured review comment on the PR

## What Gets Reviewed

The review covers all platform components with checks specific to each:

| Component | Key Checks |
|-----------|------------|
| Go Backend | Security (user token auth, RBAC), error handling, no `panic()`, K8s client patterns |
| NextJS Frontend | Zero `any` types, Shadcn UI only, React Query for data ops, `type` over `interface` |
| Go Operator | Reconciliation patterns, OwnerReferences, status management |
| Python Runner | ruff format/lint, PEP 8 |
| K8s Manifests | SecurityContext, YAML validity |

## Severity Levels

- **Blocker**: Must fix before merge (security vulnerabilities, token leaks)
- **Critical**: Should fix before merge (RBAC bypasses, `any` types, `panic()`)
- **Major**: Important to address (missing tests, architecture violations)
- **Minor**: Nice-to-have (style, docs)

## Reusable Skill

The core review logic lives in `.claude/skills/pr-reviewer/SKILL.md` and can be reused independently of this workflow. The skill contains all platform best practices and the structured review process.

## Structure

```text
pr-review/
├── .ambient/
│   └── ambient.json              # ACP workflow configuration
├── .claude/
│   ├── commands/
│   │   └── review.md             # /review command
│   └── skills/
│       └── pr-reviewer/
│           └── SKILL.md          # Reusable review skill
└── README.md
```

## Requirements

- `gh` CLI authenticated with access to the target repository

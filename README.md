# Tim's Ambient Workflows

A collection of experimental workflows for use with the [Ambient Code Platform](https://github.com/ambient-code).

## Workflows

### Activity Type Classifier

Classifies Jira issues into Red Hat's Sankey capacity allocation categories and batch-updates them via MCP Jira tools. Designed for the HCM/HCP organization's quarterly capacity reporting.

**Categories**: Associate Wellness & Development, Incidents & Support, Security & Compliance, Quality / Stability / Reliability, Future Sustainability, Product / Portfolio Work.

**How it works**:

1. Fetches issues matching a JQL query (e.g., Epics with no Activity Type set)
2. Classifies each issue using the Sankey guidance — child issues inherit from their parent when possible
3. Validates classifications and generates a markdown report with confidence levels
4. Applies updates to Jira after explicit user approval
5. Iterates — classified issues drop out of the next query, enabling batch processing

**Prerequisites**: MCP Jira server (mcp-atlassian) configured with access to `redhat.atlassian.net`.

**Usage**: Select "Activity Type Classifier" from the ACP workflow list, or load as a custom workflow pointing to `activity-type-classifier/` in this repo. See the [workflow README](activity-type-classifier/README.md) for details.

### Ambient Fork PR Review

Reviews pull requests for the Ambient Code Platform against its documented best practices, security standards, and architectural conventions. Posts review findings as a comment on the PR.

**What it reviews**:

| Component | Key Checks |
|---|---|
| Go Backend | User token auth, RBAC enforcement, error handling, no `panic()`, K8s client patterns |
| NextJS Frontend | Zero `any` types, Shadcn UI only, React Query for data ops, `type` over `interface` |
| Go Operator | Reconciliation patterns, OwnerReferences, status management |
| Python Runner | ruff format/lint, PEP 8 |
| K8s Manifests | SecurityContext, YAML validity |

Findings are classified as Blocker, Critical, Major, or Minor. The review skill in `.claude/skills/ambient-fork-pr-review/SKILL.md` can be reused independently of the workflow.

**Prerequisites**: `gh` CLI authenticated with access to the target repository.

**Usage**: Provide a PR URL or number via the `/ambient-fork-review` command. Default target repo is `tiwillia/platform`. See the [workflow README](ambient-fork-pr-review/README.md) for details.

# Activity Type Classifier

An ACP workflow that classifies Jira issues into Red Hat's Sankey Activity Type categories and batch-updates them via MCP Jira tools.

## What It Does

1. **Gathers** Jira issues matching a JQL query (default: Epics without an Activity Type)
2. **Classifies** each issue against the Sankey capacity allocation guidance
3. **Validates** classifications and generates a markdown report
4. **Applies** updates to Jira with user approval
5. **Iterates** — classified issues drop out of the next query, enabling batch processing

## Activity Types

| Category | Grouping |
|---|---|
| Associate Wellness & Development | Proactive |
| Incidents & Support | Reactive |
| Security & Compliance | Reactive |
| Quality / Stability / Reliability | Reactive |
| Future Sustainability | Proactive |
| Product / Portfolio Work | Proactive |

Categories are ordered by "what would put us out of business fastest if ignored" per the HCM/HCP Sankey guidance.

## Prerequisites

- **MCP Jira server**: The mcp-atlassian MCP server must be configured with access to `redhat.atlassian.net`
- **jq**: Required by the validation script

## Permissions

`.claude/settings.json` pre-allows all Jira MCP tools using two wildcard patterns:

- `mcp__*__jira_*` — matches Jira tools served through any ACP plugin
- `mcp__jira_*` — matches Jira tools served directly (no plugin wrapper)

## Usage

### In ACP

Select "Activity Type Classifier" from the workflow list, or load as a Custom Workflow:
- **URL**: `https://github.com/tiwillia/tims-ambient-workflows.git`
- **Branch**: `main`
- **Path**: `activity-type-classifier`

### In Claude Code

From the workflow directory, start Claude Code and invoke the skill:

```
Classify OCM Epics without activity types
```

## Directory Structure

```
activity-type-classifier/
├── .ambient/
│   └── ambient.json                        # ACP workflow config
├── .claude/
│   └── skills/
│       └── classifying-activity-types/
│           ├── SKILL.md                    # Workflow hub
│           ├── resources/
│           │   ├── activity-type-guidance.md   # Classification guidance
│           │   ├── mcp-jira-reference.md       # MCP tool instructions
│           │   └── report-template.md          # Report format
│           └── scripts/
│               ├── validate-classifications.sh
│               └── generate-report.py
├── scripts/
│   └── cleanup.sh                          # Remove data artifacts, keep reports
├── CLAUDE.md                               # Persistent context
├── README.md                               # This file
└── artifacts/activity-type-classifier/     # Output directory
```

## Outputs

| Artifact | Path |
|---|---|
| Classification Report | `artifacts/activity-type-classifier/report.md` |
| Raw Classifications | `artifacts/activity-type-classifier/classifications.json` |
| Fetched Issues | `artifacts/activity-type-classifier/issues.json` |

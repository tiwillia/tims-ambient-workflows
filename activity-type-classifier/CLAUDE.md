# Activity Type Classifier

## Jira Instance

`https://redhat.atlassian.net` (Jira Cloud)

## Custom Field

- **Name**: Activity Type
- **CLI key**: `activity-type`
- **Type**: Select list (single)

## Valid Activity Type Values

- "Associate Wellness & Development"
- "Incidents & Support"
- "Security & Compliance"
- "Quality / Stability / Reliability"
- "Future Sustainability"
- "Product / Portfolio Work"

## Critical Rules

- Never update Jira issues without explicit user approval
- Always show the classification report before offering to apply updates
- Max 2 concurrent MCP Jira operations to avoid rate limiting
- Verify MCP Jira tools are available before starting any work — stop if unavailable

## Artifact Output

`artifacts/activity-type-classifier/`

## Cleanup

After updates are applied, run `scripts/cleanup.sh` to remove collected data (issues.json, classifications.json) while preserving reports.
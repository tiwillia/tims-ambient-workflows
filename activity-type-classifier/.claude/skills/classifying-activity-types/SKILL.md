---
name: classifying-activity-types
description: Classifies Jira issues into Red Hat Sankey Activity Type categories using MCP Jira tools. Use when the user wants to batch-classify or set activity types on Jira issues, or mentions activity types, work types, Sankey, or capacity allocation.
---

# Activity Type Classification

Classify Jira issues into Red Hat's Sankey capacity allocation categories and batch-update them via MCP Jira tools.

## Valid Activity Types

These are the only valid values. Use the exact strings:

| Activity Type | Short Description |
|---|---|
| Associate Wellness & Development | Onboarding, training, AI learning, conferences, team health |
| Incidents & Support | Production incidents, customer escalations, on-call |
| Security & Compliance | CVEs, weaknesses, FedRAMP, compliance, security tooling |
| Quality / Stability / Reliability | Bugs, SLOs, chores, tech debt, toil reduction, PMR actions |
| Future Sustainability | Proactive architecture, productivity improvements, upstream, enablement |
| Product / Portfolio Work | New features, enhancements, strategic product/portfolio work |

For detailed definitions and subcategories, see [resources/activity-type-guidance.md](resources/activity-type-guidance.md).

## Activity Type Field

The Activity Type custom field ID is `customfield_10464`. This is the same across all projects.

## Prerequisites Check

Before starting any work, verify MCP Jira tools are available:

1. Attempt to call `jira_search` with a simple query (e.g., `jql: "project = OCM" limit: 1`)
2. If the tool is not found or returns an MCP connection error, **stop immediately** and tell the user:
   - "The MCP Jira tools are not available. This workflow requires the mcp-atlassian MCP server to be configured with access to redhat.atlassian.net."
   - See [resources/mcp-jira-reference.md](resources/mcp-jira-reference.md) for setup instructions
3. Do NOT proceed to any phase if this check fails

## Workflow

Copy this checklist and track progress:

```
Classification Progress:
- [ ] Prerequisites: MCP Jira tools available
- [ ] Phase 1: Gather issues from Jira
- [ ] Phase 2: Classify each issue
- [ ] Phase 3: Validate & generate report
- [ ] Phase 4: Apply updates (with approval)
- [ ] Phase 5: Iterate (optional)
```

### Phase 1: Gather Issues

Parse user input for:
- **Project key** (required) — e.g., OCM, ARO
- **Issue type** (optional, default: Epic)
- **Extra JQL filters** (optional) — e.g., `AND resolved >= "2025-01-01"`

**Always filter for issues without an Activity Type set.** The `"Activity Type" is EMPTY` condition is mandatory in every query — do not ask the user whether to include it. Issues that already have an Activity Type are out of scope.

Follow the batch fetching instructions in [resources/mcp-jira-reference.md](resources/mcp-jira-reference.md) to:
1. Construct the JQL query
2. Execute `jira_search` with pagination (up to 100 issues)
3. Save raw results to `artifacts/activity-type-classifier/issues.json`

Report the count of issues found to the user before proceeding.

### Phase 2: Classify Issues

**Pre-check — Parent inheritance**: Before classifying each issue, check if it has a parent issue. If the parent has an Activity Type set, inherit it directly — no further classification needed. Set confidence to "High" and reasoning to "Inherited from {PARENT_KEY}". See [resources/mcp-jira-reference.md](resources/mcp-jira-reference.md) for how to look up parent activity types.

For remaining issues (no parent or parent has no Activity Type), read summary, description, labels, comments, and status. Apply the classification rules below and the detailed guidance in [resources/activity-type-guidance.md](resources/activity-type-guidance.md).

Save classifications to `artifacts/activity-type-classifier/classifications.json` as a JSON array:

```json
[
  {
    "key": "OCM-12345",
    "summary": "Issue title",
    "activityType": "Product / Portfolio Work",
    "confidence": "High",
    "reasoning": "New customer-facing feature for cluster provisioning"
  },
  {
    "key": "OCM-12346",
    "summary": "Child of classified parent",
    "activityType": "Security & Compliance",
    "confidence": "High",
    "reasoning": "Inherited from OCM-11111"
  }
]
```

If total issues exceed 50, process in sub-batches of 20 to manage context.

### Phase 3: Validate and Report

1. Run the validation script:
   ```bash
   bash .claude/skills/classifying-activity-types/scripts/validate-classifications.sh artifacts/activity-type-classifier/classifications.json
   ```
2. Fix any validation errors before proceeding
3. Generate the report:
   ```bash
   python3 .claude/skills/classifying-activity-types/scripts/generate-report.py artifacts/activity-type-classifier/classifications.json artifacts/activity-type-classifier/report.md
   ```
4. Display the report summary to the user

### Phase 4: Apply Updates

1. Ask the user for **explicit approval** before modifying any Jira issues
2. If approved, follow the batch update instructions in [resources/mcp-jira-reference.md](resources/mcp-jira-reference.md)
3. Max 2 concurrent MCP update calls to avoid rate limiting
4. Report progress every 10 issues
5. On error: log the failure, continue with remaining issues
6. After completion, summarize successes and failures

### Phase 5: Iterate

After applying updates, offer to re-run the workflow:
- Classified issues will no longer match the JQL query (Activity Type is no longer empty)
- A new batch of unclassified issues can be gathered and processed
- Track cumulative stats across iterations

## Classification Rules

When classifying an issue:

1. **Read carefully** — consider summary, description, labels, comments, and linked issues
2. **Business intent over technical details** — a Kubernetes refactoring driven by product requirements is "Product / Portfolio Work", not "Future Sustainability"
3. **Security always wins** — if an issue involves CVEs, vulnerabilities, compliance, or security tooling, classify as "Security & Compliance" regardless of other aspects
4. **Primary purpose** — when an issue spans multiple categories, choose the one that best matches the primary motivation
5. **Confidence levels**:
   - **High** — clear match, unambiguous indicators
   - **Medium** — reasonable match but some ambiguity
   - **Low** — uncertain, multiple categories could apply
6. **Truncate descriptions** — use the first 2000 characters of the description for classification

For the complete category definitions with subcategories and examples, see [resources/activity-type-guidance.md](resources/activity-type-guidance.md).

## Reference Files

| File | Purpose | When to Read |
|---|---|---|
| [resources/activity-type-guidance.md](resources/activity-type-guidance.md) | Full Sankey category definitions and subcategories | Phase 2 (classifying) |
| [resources/mcp-jira-reference.md](resources/mcp-jira-reference.md) | MCP tool usage: fetch and update | Phase 1 (gather), Phase 4 (apply) |
| [resources/report-template.md](resources/report-template.md) | Report format reference | Phase 3 (report generation) |
| [scripts/validate-classifications.sh](scripts/validate-classifications.sh) | Validate classifications JSON | Phase 3 (validation) |
| [scripts/generate-report.py](scripts/generate-report.py) | Generate markdown report from JSON | Phase 3 (report generation) |

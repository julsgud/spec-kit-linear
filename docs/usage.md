# Usage Guide

This guide walks through the full spec-kit-linear workflow — from brain dump to implementation.

## Table of Contents

1. [First Time Setup](#first-time-setup)
2. [Basic Workflow](#basic-workflow)
3. [Command Details](#command-details)
4. [Configuration](#configuration)
5. [Troubleshooting](#troubleshooting)

---

## First Time Setup

### 1. Install the extension

From your spec-kit project directory:

```bash
specify extension add julsgud/spec-kit-linear
```

### 2. Configure

Edit `.specify/extensions/linear/linear-config.yml`:

```yaml
mcp_server: "claude_ai_Linear"  # your MCP server name
team: "My Team"                  # must match your Linear team exactly
project: ""                      # optional Linear project
defaults:
  labels: []
  priority: 3        # Normal
  status: "Backlog"
  inherit_labels: true
```

The `mcp_server` value depends on your setup. Common values are `claude_ai_Linear` (Claude desktop) or `linear`.

### 3. Verify Linear MCP access

Make sure your AI assistant has a Linear MCP server connected. The extension uses Linear's MCP tools to read and write issues and comments.

---

## Basic Workflow

Start with a Linear issue containing your rough requirements — notes, links, ideas. Then run commands to progressively structure it:

```
/speckit.linear.specify BOT-140       # structure the brain dump
/speckit.linear.clarify               # find gaps (optional)
/speckit.linear.analyze               # cross-check artifacts (optional)
/speckit.linear.plan                  # create implementation plan
/speckit.linear.checklist             # validate spec quality (optional)
/speckit.linear.tasks                 # create child issues from plan
/speckit.linear.implement BOT-141     # start coding a task
```

After each command, the recommended next command (with the issue ID) is copied to your clipboard — just paste to continue through the workflow.

### What gets created

All artifacts are posted as **comments** on the parent issue. The issue description is never modified.

```
BOT-140 (your issue)
├── Description: your original brain dump (untouched)
├── Comment: Specification v1
├── Comment: Clarifications v1 (if you ran clarify)
├── Comment: Analysis v1 (if you ran analyze)
├── Comment: Plan v1
├── Comment: Checklist v1 (if you ran checklist)
└── Child issues (created by /tasks):
    ├── BOT-141: T001 - First task
    ├── BOT-142: T002 - Second task
    └── BOT-143: T003 - Third task
```

---

## Command Details

### `/speckit.linear.specify <issue>`

Reads the issue description and comments, then posts a structured **Specification** comment with:
- Goals, User Stories, Acceptance Criteria
- Scope (in/out), Technical Notes
- Open Questions (if any)

**Input**: Required — Linear issue URL or identifier (e.g. `BOT-140` or full URL).

### `/speckit.linear.clarify [issue]`

Reviews the specification and posts a **Clarifications** comment identifying:
- Ambiguities, Missing Details, Edge Cases
- Dependencies, Contradictions
- Suggested clarifications with questions or defaults

**Requires**: Specification comment on the issue.

### `/speckit.linear.analyze [issue]`

Cross-artifact consistency check. Posts an **Analysis** comment with:
- Coverage assessment (how well does the spec cover the description?)
- Consistency check, Risk assessment
- Completeness gaps, Recommendations

**Requires**: Specification comment on the issue.

### `/speckit.linear.plan [issue]`

Creates a technical **Plan** comment with:
- High-level approach
- Ordered tasks (T001, T002, ...) with descriptions, acceptance criteria, and dependencies
- Testing strategy, Risks and mitigations

Uses specification + any clarifications/analysis if present.

**Requires**: Specification comment on the issue.

### `/speckit.linear.checklist [issue]`

Generates a requirements quality **Checklist** comment with interactive checkboxes across five dimensions:
- Completeness, Clarity, Consistency, Measurability, Coverage

Each item is specific to the actual spec content, with traceability refs.

**Requires**: Specification comment on the issue.

### `/speckit.linear.tasks [issue]`

Creates **child issues** in Linear from the plan's task list. This command is idempotent — re-running it skips tasks that already have matching child issues (matched by `T001:` title prefix).

Child issues inherit the parent's team and optionally labels/priority/status from config.

**Requires**: Plan comment on the issue.

### `/speckit.linear.implement [issue]`

Reads the parent issue's specification and plan, plus the target child task(s), and provides implementation guidance — what to build, where to start, dependencies, acceptance criteria, and testing approach.

Accepts either a parent issue (implements all tasks) or a child task issue (implements just that task with parent context). When implementing a single child task, automatically creates or checks out the branch from Linear's `branchName` field.

**Requires**: Specification + Plan comments on the parent issue.

---

## Configuration

Config file: `.specify/extensions/linear/linear-config.yml`

| Setting | Default | Description |
|---------|---------|-------------|
| `mcp_server` | `claude_ai_Linear` | Name of the Linear MCP server in your setup |
| `team` | (required) | Linear team name — must match exactly |
| `project` | `""` | Optional Linear project for grouping |
| `defaults.labels` | `[]` | Labels to apply to child issues |
| `defaults.priority` | `3` | Priority: 0=None, 1=Urgent, 2=High, 3=Normal, 4=Low |
| `defaults.status` | `Backlog` | Initial status for child issues |
| `defaults.inherit_labels` | `true` | Copy parent issue's labels to children |

---

## Troubleshooting

### "No Specification found on BOT-140"

Run `/speckit.linear.specify BOT-140` first. Commands depend on previous artifacts — specify must come before clarify/analyze/plan.

### "Could not parse Linear issue"

The input must be a Linear URL (`https://linear.app/.../issue/BOT-140/...`) or a bare identifier (`BOT-140`).

### "Cannot infer issue from branch"

Your git branch name doesn't contain a Linear issue identifier. Either switch to a branch like `bot-140-feature-name` or pass the issue explicitly.

### Wrong MCP server name

If Linear MCP calls fail, check that `mcp_server` in your config matches the server name in your AI assistant's MCP configuration. Run your assistant's MCP tool list to find the correct name.

### Duplicate child issues

The `/tasks` command is idempotent — it matches existing children by title prefix (`T001:`, `T002:`, etc.). If you renamed child issues and removed the prefix, `/tasks` will create duplicates. Keep the `T00N:` prefix intact.

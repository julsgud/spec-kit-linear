# spec-kit-linear

Linear integration extension for [spec-kit](https://github.com/github/spec-kit). Specs, plans, and tasks live as Linear issues and comments — Linear is the source of truth.

## How it works

You brain-dump requirements into a Linear issue, then run spec-kit commands to progressively enrich it:

```
specify  →  clarify  →  analyze  →  plan  →  tasks  →  implement
```

Each command reads the issue and previous artifacts, then posts a structured comment back to the issue. The `/tasks` command creates child issues. The `/implement` command reads everything and guides code implementation.

All artifacts are tracked via HTML comment sentinels (`<!-- speckit:specification:v1 -->`) so commands can reliably find and version them.

## Installation

Copy or symlink this extension into your project's spec-kit extensions directory:

```bash
# From your project root
mkdir -p .specify/extensions/linear
cp -r /path/to/spec-kit-linear/* .specify/extensions/linear/

# Or symlink
ln -s /path/to/spec-kit-linear .specify/extensions/linear
```

## Configuration

Copy the config template and customize:

```bash
cp .specify/extensions/linear/linear-config.template.yml .specify/extensions/linear/linear-config.yml
```

Edit `linear-config.yml`:

```yaml
mcp_server: "claude_ai_Linear"  # Your Linear MCP server name
team: "My Team"                  # Linear team name
project: ""                      # Optional project name
defaults:
  labels: []
  priority: 3          # 0=None, 1=Urgent, 2=High, 3=Normal, 4=Low
  status: "Backlog"    # Initial status for child issues
  inherit_labels: true # Copy parent labels to child issues
auto_branch: true      # Prompt to create git branch on /specify
```

Add the config file to `.gitignore` (it may contain team-specific settings):

```bash
echo ".specify/extensions/linear/linear-config.yml" >> .gitignore
```

## Linear MCP Server

This extension requires a Linear MCP server connected to your AI assistant. The tools used are:

- `get_issue` — read issue details
- `list_comments` — read comments on an issue
- `save_comment` — post/edit comments
- `save_issue` — create child issues
- `list_issues` — list child issues by parent
- `list_issue_statuses` — resolve status names to IDs
- `list_issue_labels` — resolve label names to IDs

## Commands

### `/speckit.linear.specify <issue>`

Reads a Linear issue and synthesizes a structured specification as a comment.

```
/speckit.linear.specify BOT-140
/speckit.linear.specify https://linear.app/myteam/issue/BOT-140/my-feature
```

Produces a specification with Goals, User Stories, Acceptance Criteria, Scope, Technical Notes, and Open Questions. Optionally creates a git branch.

### `/speckit.linear.clarify <issue>`

Reviews the specification to identify gaps, ambiguities, and questions.

```
/speckit.linear.clarify BOT-140
```

Requires: specification comment exists on the issue.

### `/speckit.linear.analyze <issue>`

Cross-checks all artifact comments for consistency and coverage.

```
/speckit.linear.analyze BOT-140
```

Requires: specification comment exists on the issue.

### `/speckit.linear.plan <issue>`

Creates a technical implementation plan with ordered, dependency-aware tasks.

```
/speckit.linear.plan BOT-140
```

Requires: specification comment exists on the issue.

### `/speckit.linear.tasks <issue>`

Creates Linear child issues from the plan. Idempotent — safe to re-run.

```
/speckit.linear.tasks BOT-140
```

Requires: plan comment exists on the issue. Checks existing children by title prefix (`T001:`, `T002:`) to avoid duplicates.

### `/speckit.linear.implement <issue>`

Reads parent spec/plan and child task issues, then guides implementation. Accepts either the parent issue or a specific child task.

```
/speckit.linear.implement BOT-140      # all tasks
/speckit.linear.implement BOT-141      # specific child task (resolves parent automatically)
```

Requires: specification and plan comments exist on the parent issue.

## Workflow

Typical usage:

```
1. Create a Linear issue with your raw requirements
2. /speckit.linear.specify BOT-140        → structured spec comment
3. /speckit.linear.clarify BOT-140        → gaps and questions (optional)
4. /speckit.linear.plan BOT-140           → implementation plan
5. /speckit.linear.tasks BOT-140          → child issues created
6. /speckit.linear.implement BOT-141      → guided implementation per task
```

`clarify` and `analyze` are optional — `plan` will work with just a specification, though results improve with more context.

## Versioning

Re-running a command posts a new versioned comment (v1 → v2 → v3). Previous versions are marked as superseded but preserved for audit trail.

## License

MIT

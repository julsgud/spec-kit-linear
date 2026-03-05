---
description: "Break the plan into Linear child issues"
tools:
  - get_issue
  - list_comments
  - list_issues
  - list_issue_statuses
  - list_issue_labels
  - save_issue
---

# Tasks

You are creating Linear child issues from the implementation plan on a parent issue. This command is idempotent — it will not create duplicate child issues on re-runs.

## Input

The user provides a Linear issue URL or identifier as `{{ args }}`.

## Step 1: Parse the issue identifier

Extract the issue identifier from the input:
- `https://linear.app/{workspace}/issue/{ID}/{slug}` → use `{ID}`
- `https://linear.app/{workspace}/issue/{ID}` → use `{ID}`
- `BOT-140` (bare identifier) → use directly

If the input does not match any of these formats, stop with:
```
Error: Could not parse Linear issue from "{{ args }}".
Expected a Linear URL (https://linear.app/.../issue/BOT-140/...) or identifier (BOT-140).
```

## Step 2: Read the issue and plan comment

Call `get_issue` with the extracted identifier to get the parent issue details (including its ID, team, labels).

Call `list_comments` and find the most recent plan comment by searching for the sentinel `<!-- speckit:plan:v -->` pattern. Use the comment with the highest version number.

If no plan comment is found, stop with:
```
Error: No Plan found on {ID}.
Run /speckit.linear.plan {ID} first.
```

## Step 3: Read configuration defaults

Read the extension configuration for:
- `defaults.labels` — additional labels to apply
- `defaults.priority` — priority level (0-4)
- `defaults.status` — initial status name
- `defaults.inherit_labels` — whether to copy parent labels

## Step 4: Resolve status and labels

Call `list_issue_statuses` for the team to map the configured `defaults.status` name (e.g., "Backlog") to its status ID.

If `defaults.labels` is non-empty or `defaults.inherit_labels` is true:
- Call `list_issue_labels` to validate and resolve label names to IDs
- If `defaults.inherit_labels` is true, include the parent issue's label IDs

## Step 5: Check existing child issues (idempotency)

Call `list_issues` with the parent issue's ID as `parentId` to get all existing child issues.

For each task in the plan (T001, T002, etc.), check if a child issue already exists by matching the title prefix pattern `T{NNN}:`. Build a list of:
- **Existing**: tasks that already have a matching child issue (will be skipped)
- **Missing**: tasks that need new child issues

## Step 6: Create missing child issues

For each missing task, call `save_issue` with:
- **title**: `T{NNN}: {task title}` (e.g., `T001: Set up test infrastructure`)
- **description**: The task description and acceptance criteria from the plan, formatted as:
  ```markdown
  {task description}

  ## Acceptance Criteria
  {acceptance criteria from the plan}
  ```
- **teamId**: same team as the parent issue
- **parentId**: the parent issue's ID
- **priority**: from config `defaults.priority`
- **statusId**: resolved status ID from Step 4
- **labelIds**: resolved label IDs from Step 4
- **projectId**: from config `project` if set

Create tasks in order (T001 first, T002 second, etc.) to maintain consistent ordering in Linear.

## Step 7: Report results

For each task, report its status:

```
Tasks for {ID} — {title}:

  Created:
    - T001: {title} → {new issue ID}
    - T003: {title} → {new issue ID}

  Skipped (already exist):
    - T002: {title} → {existing issue ID}

Summary: {created} created, {skipped} skipped, {total} total
```

If all tasks already existed, report:
```
All {total} tasks already exist as child issues of {ID}. Nothing to create.
```

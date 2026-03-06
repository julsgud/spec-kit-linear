---
description: "Guide implementation from Linear task issues"
tools:
  - get_issue
  - list_comments
  - list_issues
---

# Implement

You are guiding the implementation of work defined in Linear task issues. This command reads the parent issue's specification and plan, plus the child task issues, to provide full context for implementation.

## Input

The user provides a Linear issue URL or identifier as `{{ args }}`. If no argument is provided, the issue is inferred from the current git branch name. This can be either:
- A **parent** issue (has child task issues) — will implement all tasks
- A **child** task issue (has a parent) — will implement just that task, with parent context

## Step 1: Parse the issue identifier

**If `{{ args }}` is provided**, extract the issue identifier from the input:
- `https://linear.app/{workspace}/issue/{ID}/{slug}` → use `{ID}`
- `https://linear.app/{workspace}/issue/{ID}` → use `{ID}`
- `BOT-140` (bare identifier) → use directly

If the input does not match any of these formats, stop with:
```
Error: Could not parse Linear issue from "{{ args }}".
Expected a Linear URL (https://linear.app/.../issue/BOT-140/...) or identifier (BOT-140).
```

**If `{{ args }}` is empty**, infer the issue from the current git branch:
1. Run `git branch --show-current` to get the branch name.
2. If the result is empty (detached HEAD), stop with:
   ```
   Error: Cannot infer issue — HEAD is detached. Provide an issue identifier explicitly.
   ```
3. If the branch is `main`, `master`, or `develop`, stop with:
   ```
   Error: Cannot infer issue from default branch "{branch}". Provide an issue identifier explicitly.
   ```
4. Extract the first match of the pattern `[A-Za-z]+-[0-9]+` from the branch name and uppercase it to get the issue identifier.
5. If no match is found, stop with:
   ```
   Error: Cannot infer issue from branch "{branch}" — no identifier found. Provide an issue identifier explicitly.
   ```
6. Print: `Inferred issue {ID} from branch "{branch}".`

## Step 2: Resolve the parent issue

Call `get_issue` with the extracted identifier.

Determine if this is a parent or child issue:
- If the issue **has a parent** → this is a child task issue. Call `get_issue` on the parent to get the parent issue.
- If the issue **has no parent** → this is the parent issue itself.

Store both:
- `parent_issue`: the parent issue (with the specification and plan)
- `target_issues`: the specific task issue(s) to implement. If a child was given, this is just that child. If the parent was given, this is all children.

## Step 3: Read parent artifacts

Call `list_comments` on the parent issue and find:
- The most recent specification comment (`<!-- speckit:specification:v* -->`) — **required**
- The most recent plan comment (`<!-- speckit:plan:v* -->`) — **required**
- The most recent clarifications comment (`<!-- speckit:clarifications:v* -->`) — optional
- The most recent analysis comment (`<!-- speckit:analysis:v* -->`) — optional

If no specification is found, stop with:
```
Error: No Specification found on {parent ID}.
Run /speckit.linear.specify {parent ID} first.
```

If no plan is found, stop with:
```
Error: No Plan found on {parent ID}.
Run /speckit.linear.plan {parent ID} first.
```

## Step 4: Read child task issues

If the parent was given directly (implementing all tasks):
- Call `list_issues` with `parentId` to get all child issues
- Sort by title prefix (T001 < T002 < T003)

If a child was given (implementing a single task):
- Also call `list_issues` with `parentId` to see sibling tasks for context
- Mark which task is the current focus

## Step 5: Present implementation context

Present the gathered context in a structured format:

```
## Implementation Context

### Parent Issue: {parent ID} — {parent title}

### Specification (v{N})
{specification content}

### Plan (v{N})
{plan content}

{if clarifications exist:}
### Clarifications (v{N})
{clarifications content}

{if analysis exists:}
### Analysis (v{N})
{analysis content}

### Task(s) to Implement
{for each target task:}
#### {task ID}: {task title}
Status: {status}
Priority: {priority}
Description:
{task description}
```

## Step 6: Guide implementation

Based on the context, provide implementation guidance:

1. **What to build**: Summarize the specific work for the target task(s)
2. **Where to start**: Suggest which files to create or modify based on the plan
3. **Dependencies**: Note which sibling tasks must be completed first (check their status)
4. **Acceptance criteria**: Restate the specific acceptance criteria to verify against
5. **Testing**: What tests to write, based on the plan's testing strategy

If implementing a single task and its dependencies (other task IDs) are not yet completed, warn:
```
Warning: Task {ID} depends on {dep IDs} which are not yet completed.
Consider implementing dependencies first, or proceed with awareness of the dependency.
```

## Output

After presenting context and guidance, ask the user how they want to proceed:
- Implement the task(s) now (begin writing code)
- Review the plan first
- Switch to a different task

---
description: "Create a technical implementation plan from the specification"
tools:
  - get_issue
  - list_comments
  - list_issues
  - save_comment
---

# Plan

You are creating a technical implementation plan from the specification and any other artifacts on a Linear issue.

## Input

The user provides a Linear issue URL or identifier as `{{ args }}`. If no argument is provided, the issue is inferred from the current git branch name.

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

## Step 2: Read the issue and all artifact comments

Call `get_issue` with the extracted identifier.

Call `list_comments` and collect all spec-kit artifact comments by searching for sentinels:
- `<!-- speckit:specification:v* -->` — Specification (**required**)
- `<!-- speckit:clarifications:v* -->` — Clarifications (optional)
- `<!-- speckit:analysis:v* -->` — Analysis (optional)

For each artifact type, use the comment with the highest version number.

If no specification comment is found, stop with:
```
Error: No Specification found on {ID}.
Run /speckit.linear.specify {ID} first.
```

## Step 3: Check for existing plans

Search the comments for existing plan sentinels (`<!-- speckit:plan:v* -->`). If previous versions exist, note the highest version number.

## Step 4: Synthesize the implementation plan

Using the specification (and clarifications/analysis if present), create a plan with:

### Approach
High-level technical approach. What architectural pattern, strategy, or design will be used? Explain the "how" at a conceptual level.

### Tasks

Break the work into ordered, implementable tasks. Each task should be:
- **Small enough** to be a single focused unit of work (a few hours to a day)
- **Large enough** to be meaningful (not individual lines of code)
- **Ordered** by dependency — tasks that unblock others come first

For each task, provide:
- **ID**: `T001`, `T002`, etc.
- **Title**: concise description
- **Description**: what needs to be done, specific enough to implement from
- **Acceptance criteria**: how to verify this task is complete
- **Dependencies**: which other task IDs must be done first (if any)

### Testing Strategy
How should the implementation be tested? What types of tests (unit, integration, e2e) are needed and for which tasks?

### Risks and Mitigations
Technical risks identified during planning and how to mitigate them. Incorporate any risks from the analysis artifact if present.

## Step 5: Resolve risks and validate tasks (autonomous resolution)

Read the `resolution.enabled` setting from the extension config. If `false`, skip this step entirely.

For each item in the **Risks and Mitigations** section and each task in the **Tasks** section, dispatch sub-agents (using the Agent tool) to research and validate. Respect the `resolution.max_agents` config cap.

Each sub-agent should research using the strategies defined in `resolution.strategies` (in order):
1. **codebase** — use Glob, Grep, and Read to estimate complexity, find existing code to build on, and validate task scope
2. **linear** — use `list_issues` to check for related work or prior implementations
3. **web** — use WebSearch and WebFetch to find architectural patterns, library recommendations, and risk mitigations

For **Risks and Mitigations**, each sub-agent must return exactly one of:
- `RESOLVED: {mitigation}` — concrete mitigation strategy found with supporting evidence
- `SUGGESTION: {recommendation}` — potential mitigation, but needs validation
- `NEEDS-HUMAN: {reason}` — risk requires human judgment (e.g., business decision, security review)

For **Tasks**, sub-agents validate:
- Whether existing code can be leveraged (note specific files/functions)
- Whether the task's scope estimate is reasonable
- Whether dependencies between tasks are correctly identified

After all sub-agents complete, annotate findings inline in the plan artifact:
- Risk resolutions: append `→ **Resolved:** {text} _(auto-resolved via {source})_`
- Risk suggestions: append `→ **Suggested:** {text} _(confidence: medium)_`
- Risk needs human: append `→ **Needs human input:** {reason}`
- Task validations: append `→ **Validated:** {notes} _(auto-validated via {source})_`

Filter results based on `resolution.auto_resolve_threshold`:
- `"high"` — only RESOLVED items are treated as resolved
- `"medium"` — both RESOLVED and SUGGESTION items are treated as resolved

## Step 6: Post the plan comment

Determine the version number:
- If no previous plan comment exists → `v1`
- If previous versions exist → increment

Format the comment body as:

```markdown
<!-- speckit:plan:v{N} -->
## Plan

> Generated by spec-kit-linear | v{N} | {today's date}

### Approach
{approach}

### Tasks

#### T001: {title}
{description}

**Acceptance criteria:** {criteria}
**Dependencies:** {none or list of task IDs}

#### T002: {title}
{description}

**Acceptance criteria:** {criteria}
**Dependencies:** {task IDs}

{...more tasks as needed}

### Testing Strategy
{testing strategy}

### Risks and Mitigations
{risks}
```

Call `save_comment` to post this as a new comment on the issue.

If a previous version exists, edit the previous version's comment to prepend `**(superseded by v{N})**` on the line immediately after the `## Plan` header.

## Output

Summarize what was done:
- Issue: `{ID} — {title}`
- Artifacts used: list which artifacts informed the plan
- Plan version: `v{N}`
- Number of tasks: {count}
- Task list: brief one-line summary of each task
- Resolution: {N} auto-resolved, {N} suggestions, {N} needs-human

Then suggest the next step:

```
Next steps — pick one:
  /speckit.linear.checklist  — validate requirements quality before tasks
  /speckit.linear.tasks      — create Linear child issues from this plan
```

Then copy the recommended next command to the user's clipboard. Detect the platform and use the appropriate command:
- macOS: `printf '%s' '/speckit.linear.tasks {ID}' | pbcopy`
- Linux: `printf '%s' '/speckit.linear.tasks {ID}' | xclip -selection clipboard`
- Windows: `printf '%s' '/speckit.linear.tasks {ID}' | clip`

Print: `Copied to clipboard: /speckit.linear.tasks {ID}`

---
description: "Identify gaps and questions in the specification"
tools:
  - get_issue
  - list_comments
  - list_issues
  - save_comment
---

# Clarify

You are reviewing a specification on a Linear issue to identify gaps, ambiguities, and questions that need answers before implementation.

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

## Step 2: Read the issue and comments

Call `get_issue` with the extracted identifier.

Call `list_comments` and find the most recent specification comment by searching for the sentinel `<!-- speckit:specification:v -->` pattern. Use the comment with the highest version number.

If no specification comment is found, stop with:
```
Error: No Specification found on {ID}.
Run /speckit.linear.specify {ID} first.
```

## Step 3: Check for existing clarifications

Search the comments for existing clarification sentinels (`<!-- speckit:clarifications:v* -->`). If previous versions exist, note the highest version number. The new clarifications will be the next version.

## Step 4: Analyze for gaps and questions

Review the specification carefully and identify:

### Ambiguities
Requirements that could be interpreted in multiple ways. For each, explain the ambiguity and suggest possible interpretations.

### Missing Details
Areas where the specification lacks enough detail for implementation. What decisions need to be made?

### Edge Cases
Scenarios not addressed by the specification that could cause issues during implementation.

### Dependencies
External dependencies, services, or prerequisites that are implied but not explicitly stated.

### Contradictions
Any requirements that conflict with each other or with known constraints.

### Suggested Clarifications
For each gap identified, propose a specific question to ask the stakeholder, or suggest a reasonable default if one exists.

## Step 5: Resolve findings (autonomous resolution)

Read the `resolution.enabled` setting from the extension config. If `false`, skip this step entirely.

For each finding across all six sections (Ambiguities, Missing Details, Edge Cases, Dependencies, Contradictions, Suggested Clarifications), dispatch sub-agents (using the Agent tool) to research resolutions. Batch related findings together. Respect the `resolution.max_agents` config cap.

Each sub-agent should research using the strategies defined in `resolution.strategies` (in order), with section-specific focus:

- **Ambiguities** → search codebase for existing patterns that resolve the ambiguity; search web for best practices
- **Missing Details** → search codebase for existing implementations that fill the gap
- **Edge Cases** → search web for common patterns in similar systems; search codebase for existing error handling
- **Dependencies** → search codebase (package.json, imports, configs) for existing dependencies; search web for library documentation
- **Contradictions** → cross-reference the issue description, specification, and related Linear issues for the intended behavior
- **Suggested Clarifications** → convert vague suggestions into concrete defaults via codebase conventions and web best practices

Each sub-agent must return exactly one of:
- `RESOLVED: {answer}` — confident answer found with supporting evidence
- `SUGGESTION: {recommendation}` — reasonable default or recommendation, but not definitive
- `NEEDS-HUMAN: {reason}` — cannot be resolved autonomously; explain why

After all sub-agents complete, annotate each finding inline in the clarifications artifact:
- Resolved: append `→ **Resolved:** {text} _(auto-resolved via {source})_`
- Suggested: append `→ **Suggested:** {text} _(confidence: medium)_`
- Needs human: append `→ **Needs human input:** {reason}`

Filter results based on `resolution.auto_resolve_threshold`:
- `"high"` — only RESOLVED items are treated as resolved
- `"medium"` — both RESOLVED and SUGGESTION items are treated as resolved

## Step 6: Post the clarifications comment

Determine the version number:
- If no previous clarifications comment exists → `v1`
- If previous versions exist → increment

Format the comment body as:

```markdown
<!-- speckit:clarifications:v{N} -->
## Clarifications

> Generated by spec-kit-linear | v{N} | {today's date}

### Ambiguities
{ambiguities, or "None identified."}

### Missing Details
{missing details, or "None identified."}

### Edge Cases
{edge cases, or "None identified."}

### Dependencies
{dependencies, or "None identified."}

### Contradictions
{contradictions, or "None identified."}

### Suggested Clarifications
{numbered list of questions or suggested defaults}
```

Call `save_comment` to post this as a new comment on the issue.

If a previous version exists, edit the previous version's comment to prepend `**(superseded by v{N})**` on the line immediately after the `## Clarifications` header.

## Output

Summarize what was done:
- Issue: `{ID} — {title}`
- Specification version reviewed: `v{M}`
- Clarifications version: `v{N}`
- Number of items identified per category
- Resolution: {N} auto-resolved, {N} suggestions, {N} needs-human

Then suggest the next step:

```
Next steps — pick one:
  /speckit.linear.analyze    — cross-artifact consistency check
  /speckit.linear.plan       — create implementation plan
  /speckit.linear.specify    — re-run specify to address clarification findings
```

Then copy the recommended next command to the user's clipboard. Detect the platform and use the appropriate command:
- macOS: `printf '%s' '/speckit.linear.analyze {ID}' | pbcopy`
- Linux: `printf '%s' '/speckit.linear.analyze {ID}' | xclip -selection clipboard`
- Windows: `printf '%s' '/speckit.linear.analyze {ID}' | clip`

Print: `Copied to clipboard: /speckit.linear.analyze {ID}`

# spec-kit-linear

Linear integration extension for [spec-kit](https://github.com/github/spec-kit). Specs, plans, and tasks live as Linear issues and comments — Linear is the source of truth.

## How it works

Brain-dump requirements into a Linear issue, then run commands to progressively enrich it:

```
specify → clarify → analyze → plan → checklist → tasks → implement
```

Each command reads the issue and previous artifacts, then posts a versioned comment back. Re-running a command creates a new version (v1 → v2 → v3) — previous versions are preserved.

## Installation

```bash
# Symlink into your project's spec-kit extensions
mkdir -p .specify/extensions
ln -s /path/to/spec-kit-linear .specify/extensions/linear
```

Requires a Linear MCP server connected to your AI assistant.

## Commands

All commands accept a Linear issue URL or identifier (e.g. `BOT-140`). For all commands except `specify`, if no argument is given the issue is inferred from the current git branch.

| Command | What it does | Requires |
|---------|-------------|----------|
| `/speckit.linear.specify` | Synthesize a structured specification | — |
| `/speckit.linear.clarify` | Identify gaps and questions | specification |
| `/speckit.linear.analyze` | Cross-artifact consistency check | specification |
| `/speckit.linear.plan` | Technical implementation plan with ordered tasks | specification |
| `/speckit.linear.checklist` | Requirements quality checklist (interactive checkboxes) | specification |
| `/speckit.linear.tasks` | Create Linear child issues from the plan (idempotent) | plan |
| `/speckit.linear.implement` | Guide code implementation per task | specification + plan |

`clarify`, `analyze`, and `checklist` are optional — `plan` works with just a specification, though results improve with more context.

## License

MIT

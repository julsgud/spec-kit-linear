# spec-kit-linear

[![CI](https://github.com/julsgud/spec-kit-linear/actions/workflows/ci.yml/badge.svg)](https://github.com/julsgud/spec-kit-linear/actions/workflows/ci.yml)
[![spec-kit extension](https://img.shields.io/badge/spec--kit-extension-blue)](https://github.com/github/spec-kit)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Latest Release](https://img.shields.io/github/v/release/julsgud/spec-kit-linear)](https://github.com/julsgud/spec-kit-linear/releases)

Linear integration extension for [spec-kit](https://github.com/github/spec-kit). Specs, plans, and tasks live as Linear issues and comments — Linear is the source of truth.

## How it works

Brain-dump requirements into a Linear issue, then run commands to progressively enrich it:

```
specify → clarify → analyze → plan → checklist → tasks → implement
```

Each command reads the issue and previous artifacts, then posts a versioned comment back. Re-running a command creates a new version (v1 → v2 → v3) — previous versions are preserved.

### Clipboard handoff

Every command copies the recommended next command (with the issue ID) to your clipboard. Just paste to continue through the workflow — no need to remember command names or re-type identifiers.

### Branching

Branches are created at `implement` time (not during `specify`), using the `branchName` that Linear auto-generates for each child task issue. This means no empty branches sitting around before any code is written.

## Installation

```bash
specify extension add julsgud/spec-kit-linear
```

Requires a Linear MCP server connected to your AI assistant.

## Commands

All commands accept a Linear issue URL or identifier (e.g. `BOT-140`). If no argument is given, the issue is inferred from the current git branch.

| Command | What it does | Requires |
|---------|-------------|----------|
| `/speckit.linear.specify` | Synthesize a structured specification | — |
| `/speckit.linear.clarify` | Identify gaps and questions | specification |
| `/speckit.linear.analyze` | Cross-artifact consistency check | specification |
| `/speckit.linear.plan` | Technical implementation plan with ordered tasks | specification |
| `/speckit.linear.checklist` | Requirements quality checklist (interactive checkboxes) | specification |
| `/speckit.linear.tasks` | Create Linear child issues from the plan (idempotent) | plan |
| `/speckit.linear.implement` | Create branch from Linear + guide code implementation | specification + plan |

`clarify`, `analyze`, and `checklist` are optional — `plan` works with just a specification, though results improve with more context.

## Documentation

See [docs/usage.md](docs/usage.md) for the full usage guide — setup, workflow walkthrough, configuration reference, and troubleshooting.

## License

MIT

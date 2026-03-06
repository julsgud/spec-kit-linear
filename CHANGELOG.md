# Changelog

## [1.2.0] - 2026-03-06

### Added

- Next-step guidance in all command outputs — each command suggests what to run next

## [1.1.0] - 2026-03-06

### Added

- CI workflow validating extension manifest, command frontmatter, and required files
- Usage guide at `docs/usage.md` with full workflow, configuration reference, and troubleshooting
- README badges (CI status, spec-kit extension, license, latest release)
- `.editorconfig` for consistent formatting

## [1.0.0] - 2026-03-05

### Added

- Initial release of spec-kit-linear
- Seven commands: specify, clarify, analyze, plan, checklist, tasks, implement
- Linear-first workflow: issues and comments as source of truth
- Sentinel-based comment discovery with version tracking
- Idempotent child issue creation via /tasks
- Configurable team, project, labels, priority, and status defaults
- Optional git branch creation on /specify

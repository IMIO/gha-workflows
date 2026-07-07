# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repository contains **reusable GitHub Actions workflows** for the IMIO organization. Other repositories consume these via:

```yaml
uses: IMIO/gha-workflows/.github/workflows/<workflow>.yml@v1
```

This repo has no build system, tests, or runtime code — it is purely workflow YAML definitions.

## Workflows

All workflows use `on: workflow_call` and are located in `.github/workflows/`:

| File | Purpose |
|------|---------|
| `package-test-uv.yml` | Modern Plone package testing via `uv` (default Python 3.13) |
| `package-test-legacy.yml` | Legacy Plone package testing (default Python 2.7, Plone 4.3) |
| `package-test-coverage.yml` | Plone testing with coverage reporting (optional Coveralls) |
| `package-full-test.yml` | Full pipeline: code analysis + test matrix + coverage (nests the two workflows above) |
| `promote-staging-to-production.yml` | Docker image promotion staging→prod + Rundeck deployment |
| `release.yml` | Auto-creates GitHub releases from pushed `v*` tags using CHANGELOG.md |

## Architecture

### External Actions Used
These workflows delegate to shared IMIO actions (in the separate `IMIO/gha` repository):
- `IMIO/gha/plone-package-test-notify@v5` — modern test runner with Mattermost notification
- `IMIO/gha/plone-package-test-notify@v4` — legacy test runner
- `IMIO/gha/tag-notify@v5` — Docker registry image tagging
- `IMIO/gha/rundeck-notify@v5` — Rundeck job execution

### Common Patterns
- All workflows accept a `mattermost_webhook_url` secret for failure notifications
- Most workflows accept a `runner_label` input for self-hosted runner targeting
- The `promote-staging-to-production.yml` workflow has two jobs: `tag` (retags Docker image) then `deploy` (triggers Rundeck), with conditional scheduling (quick_release = 2 min delay vs normal = next-day)
- `package-test-legacy.yml` skips draft pull requests; others do not

### Versioning
- Semantic versioning with a floating major tag (e.g., `v1` always points to latest `v1.x.x`)
- The `release.yml` workflow handles this automatically on tag push
- CHANGELOG.md is parsed to populate GitHub release notes

## Development Notes

- Use conventional commits: `feat:`, `fix:`, `chore:`, `doc:`
- To release: create and push a `vX.Y.Z` tag — the `release.yml` workflow handles the rest
- When adding inputs to a workflow, check whether they should be `required: false` with a sensible default to maintain backward compatibility for callers

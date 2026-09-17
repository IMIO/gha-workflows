# Changelog

## [v1.6.0] - 2026-09-17
### Added
- package-test-legacy.yml
    - optional `environment_variables` input, exported to `$GITHUB_ENV` before the test step, so callers can pass arbitrary environment variables (e.g. `PM_BRANCH_NAME` read by `buildout.environ` in `Products.Meeting*` sources) to buildout and to the test command


## [v1.5.0] - 2026-09-17
### Changed
- package-test-legacy.yml
    - optional `gh_token` secret so `mr.developer` source checkouts are authenticated instead of anonymous

## [v1.4.0] - 2026-09-03
### Added
- package-test-uv.yml, package-test-coverage.yml, package-full-test.yml
    - optional `gh_token` secret, forwarded to `plone-package-test-notify` as its new `GITHUB_TOKEN` input, so `mr.developer` source checkouts are authenticated instead of anonymous
    - falls back to the job's own `GITHUB_TOKEN` when the secret is omitted, so public sources are covered without any caller change. A `workflow_call` secret cannot be named `github_token` (reserved), hence `gh_token`

### Changed
- package-test-uv.yml, package-test-coverage.yml, package-full-test.yml
    - `permissions: contents: read` (was `{}`), required for the fallback `GITHUB_TOKEN` to authenticate git

## [v1.3.0] - 2026-08-31
### Added
- deb-build-push-notify.yml
    - build, sign and push a deb package to the bookworm and/or trixie apt repository, with Mattermost notification
    - target distributions selected with the `distributions` input
    - all secrets optional: fall back to the iMio naming convention, so callers only need `secrets: inherit`
    - branch aware target: `test_branch` (default `dev-test`) publishes to `NEXUS_<DISTRIBUTION>_TEST_URL`, other refs to `NEXUS_<DISTRIBUTION>_URL`

## [v1.2.2] - 2026-07-08
### Fixed
- package-full-test.yml
    - UTC timezone by default

## [v1.2.1] - 2026-07-08
### Fixed
- package-test-coverage.yml, package-test-uv.yml
    - UTC timezone by default

## [v1.2.0] - 2026-07-07
### Added
- package-full-test.yml
    - full pipeline combining code analysis, test matrix (package-test-uv.yml) and coverage (package-test-coverage.yml)

## [v1.1.2] - 2026-07-07
### Fixed
- package-test-coverage.yml
    - install coverage inside venv

## [v1.1.1] - 2026-04-15
### Fixed
- package-test-coverage.yml
    - generate coverage xml report

## [v1.1.0] - 2026-04-14
### Changed
Security hardening


## [v1.0.0] - 2025-07-17
### Added
- package-test-coverage.yml
- package-test-legacy.yml
- package-test-uv.yml
- promote-staging-to-production.yml
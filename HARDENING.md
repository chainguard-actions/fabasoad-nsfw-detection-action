<!-- markdownlint-disable -->

# Hardening Report: fabasoad--nsfw-detection-action/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--nsfw-detection-action/v3.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions and reusable workflows using mutable tags or branch names instead of full 40-character SHA commit digests, making them vulnerable to supply-chain attacks. Failing references:
- functional-tests.yml: `actions/checkout@v4`
- linting.yml: `fabasoad/reusable-workflows/.github/workflows/wf-js-lint.yml@main`, `fabasoad/reusable-workflows/.github/workflows/wf-pre-commit.yml@main`
- release.yml: `fabasoad/reusable-workflows/.github/workflows/wf-github-release.yml@main`
- security.yml: `fabasoad/reusable-workflows/.github/workflows/wf-security-sast.yml@main`
- sync-labels.yml: `fabasoad/reusable-workflows/.github/workflows/wf-sync-labels.yml@main`
- unit-tests.yml: `fabasoad/reusable-workflows/.github/workflows/wf-js-unit-tests.yml@main`
- update-license.yml: `fabasoad/reusable-workflows/.github/workflows/wf-update-license.yml@main`

Locations:

- `.github/workflows/functional-tests.yml:38`
- `.github/workflows/linting.yml:10`
- `.github/workflows/linting.yml:13`
- `.github/workflows/release.yml:10`
- `.github/workflows/security.yml:12`
- `.github/workflows/sync-labels.yml:10`
- `.github/workflows/unit-tests.yml:11`
- `.github/workflows/update-license.yml:10`

### missing-permissions (severity: medium)

Six workflow files have no top-level `permissions:` block and no job-level `permissions:` block on any of their jobs. Without explicit permissions, workflows run with the default (potentially broad) token permissions. Affected files: functional-tests.yml, linting.yml, release.yml, sync-labels.yml, unit-tests.yml, update-license.yml.

Locations:

- `.github/workflows/functional-tests.yml:1`
- `.github/workflows/linting.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/sync-labels.yml:1`
- `.github/workflows/unit-tests.yml:1`
- `.github/workflows/update-license.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions

**Notes:**

Pinned all unpinned action references to full SHA digests: actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4, and all fabasoad/reusable-workflows@main references → @10062f8186847226cb4865efbb8047795d372bae # main (across functional-tests.yml, linting.yml, release.yml, security.yml, sync-labels.yml, unit-tests.yml, update-license.yml). Added top-level `permissions: {}` block to the 6 workflow files that were missing it (functional-tests.yml, linting.yml, release.yml, sync-labels.yml, unit-tests.yml, update-license.yml). security.yml already had job-level permissions and was excluded from the missing-permissions finding.


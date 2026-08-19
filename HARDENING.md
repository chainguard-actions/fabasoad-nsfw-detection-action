<!-- markdownlint-disable -->

# Hardening Report: fabasoad--nsfw-detection-action/v2.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--nsfw-detection-action/v2.0.4** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Prepare API key' step in functional-tests.yml directly interpolates GitHub Actions expressions inside a `run:` shell block. Specifically, `${{ matrix.provider }}`, `${{ secrets[matrix.api-user] }}`, and `${{ secrets[matrix.api-key] }}` are embedded directly in shell commands. The `matrix.*` context is workflow-controllable and flows through YAML template substitution before the shell processes it, enabling script injection. Offending lines:
  `if [ "${{ matrix.provider }}" = "sightengine" ]; then`
  `echo "api-key=${{ secrets[matrix.api-user] }},${{ secrets[matrix.api-key] }}" >> $GITHUB_OUTPUT`
  `echo "api-key=${{ secrets[matrix.api-key] }}" >> $GITHUB_OUTPUT`

Locations:

- `.github/workflows/functional-tests.yml:33`

### github-env-injection (severity: high)

The 'Prepare API key' step writes values derived from `${{ secrets[matrix.api-key] }}` and `${{ secrets[matrix.api-user] }}` directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). This allows newline injection into the output file, which can be exploited to inject arbitrary key-value pairs into the GitHub Actions output context. Offending lines:
  `echo "api-key=${{ secrets[matrix.api-user] }},${{ secrets[matrix.api-key] }}" >> $GITHUB_OUTPUT`
  `echo "api-key=${{ secrets[matrix.api-key] }}" >> $GITHUB_OUTPUT`

Locations:

- `.github/workflows/functional-tests.yml:34`

### unpinned-uses (severity: high)

Multiple workflow files reference actions and reusable workflows using mutable tags or branch names instead of pinned 40-character commit SHAs. This exposes the workflows to supply-chain attacks if the referenced tag or branch is updated or hijacked. Failing references:
- functional-tests.yml: `actions/checkout@v4`
- linting.yml: `fabasoad/reusable-workflows/.github/workflows/wf-js-lint.yml@main`, `fabasoad/reusable-workflows/.github/workflows/wf-pre-commit.yml@main`
- release.yml: `fabasoad/reusable-workflows/.github/workflows/wf-github-release.yml@main`
- security.yml: `fabasoad/reusable-workflows/.github/workflows/wf-security-sast.yml@main`
- sync-labels.yml: `fabasoad/reusable-workflows/.github/workflows/wf-sync-labels.yml@main`
- unit-tests.yml: `actions/checkout@v4`, `actions/cache@v4`, `paambaati/codeclimate-action@v6.0.0`
- update-license.yml: `fabasoad/reusable-workflows/.github/workflows/wf-update-license.yml@main`

Locations:

- `.github/workflows/functional-tests.yml:30`
- `.github/workflows/linting.yml:10`
- `.github/workflows/linting.yml:13`
- `.github/workflows/release.yml:9`
- `.github/workflows/security.yml:11`
- `.github/workflows/sync-labels.yml:11`
- `.github/workflows/unit-tests.yml:18`
- `.github/workflows/unit-tests.yml:21`
- `.github/workflows/unit-tests.yml:36`
- `.github/workflows/update-license.yml:10`

### missing-permissions (severity: medium)

Six workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may include `write` access to contents and other scopes), violating the principle of least privilege. Affected files: functional-tests.yml, linting.yml, release.yml, sync-labels.yml, unit-tests.yml, update-license.yml. (security.yml is the only file with a job-level permissions block.)

Locations:

- `.github/workflows/functional-tests.yml:1`
- `.github/workflows/linting.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/sync-labels.yml:1`
- `.github/workflows/unit-tests.yml:1`
- `.github/workflows/update-license.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings across seven workflow files:

1. **script-injection** (functional-tests.yml): Moved `${{ matrix.provider }}`, `${{ secrets[matrix.api-user] }}`, and `${{ secrets[matrix.api-key] }}` from the `run:` shell block into the step's `env:` block as `PROVIDER`, `API_USER_SECRET`, and `API_KEY_SECRET`. Shell script now references plain environment variables.

2. **github-env-injection** (functional-tests.yml): Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing values to `$GITHUB_OUTPUT` to prevent newline injection.

3. **unpinned-uses**: Pinned all action references to full 40-char SHAs: `actions/checkout@v4` → `@11d5960a...`, `actions/cache@v4` → `@0057852b...`, `paambaati/codeclimate-action@v6.0.0` → `@b74bb25d...`, all `fabasoad/reusable-workflows@main` → `@5ebe0938...`.

4. **missing-permissions**: Added `permissions: {}` top-level block to functional-tests.yml, linting.yml, release.yml, sync-labels.yml, unit-tests.yml, and update-license.yml. security.yml already had appropriate job-level permissions and was left unchanged except for pinning its reusable workflow reference.


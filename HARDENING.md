<!-- markdownlint-disable -->

# Hardening Report: fabasoad--nsfw-detection-action/v3.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--nsfw-detection-action/v3.0.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions or reusable workflows using mutable tags or branch names instead of pinned 40-character commit SHAs. This exposes the action to supply-chain attacks if the referenced ref is updated maliciously.

Failing references:
- functional-tests.yml: `actions/checkout@v7` (tag)
- linting.yml: `fabasoad/reusable-workflows/.github/workflows/wf-pre-commit.yml@main` (branch)
- release.yml: `fabasoad/reusable-workflows/.github/workflows/wf-github-release.yml@main` (branch)
- security.yml: `fabasoad/reusable-workflows/.github/workflows/wf-security-sast.yml@main` (branch)
- sync-labels.yml: `fabasoad/reusable-workflows/.github/workflows/wf-sync-labels.yml@main` (branch)
- update-license.yml: `fabasoad/reusable-workflows/.github/workflows/wf-update-license.yml@main` (branch)

Locations:

- `.github/workflows/functional-tests.yml:38`
- `.github/workflows/linting.yml:13`
- `.github/workflows/release.yml:12`
- `.github/workflows/security.yml:21`
- `.github/workflows/sync-labels.yml:13`
- `.github/workflows/update-license.yml:12`

### script-injection (severity: high)

Rule (b) violation: In action.yml, the four 'Classify *' steps each construct a shell command using the unquoted variable `${INPUT_PROVIDER}`, which is sourced from `inputs.provider` (an untrusted caller-controlled input). The unquoted expansion `${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh` allows an attacker to inject shell metacharacters (e.g. `;`, `|`, `$(...)`) via the `provider` input, leading to arbitrary command execution. All four classify steps share this pattern:
  `${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh \`
The fix is to double-quote the variable: `"${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh"`.

Locations:

- `action.yml:86`
- `action.yml:101`
- `action.yml:116`
- `action.yml:131`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed script injection in action.yml by double-quoting the provider path in all four 'Classify *' steps: '"${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh"'. Fixed unpinned-uses in all six workflow files: pinned actions/checkout@v7 to SHA 3d3c42e5aac5ba805825da76410c181273ba90b1, and pinned all five fabasoad/reusable-workflows@main references to SHA ecce8eb77aa808e37355819f2a4ac62deefd0aff.


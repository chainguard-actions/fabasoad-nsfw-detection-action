<!-- markdownlint-disable -->

# Hardening Report: fabasoad--nsfw-detection-action/v2.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--nsfw-detection-action/v2.0.3** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): ${{ }} expressions are directly interpolated inside run: shell commands. In the 'Prepare API key' step, `${{ matrix.provider }}`, `${{ secrets[matrix.api-user] }}`, and `${{ secrets[matrix.api-key] }}` are embedded directly in the shell script. An attacker who can control matrix values or influence the context can inject arbitrary shell commands. Offending lines: `if [ "${{ matrix.provider }}" = "sightengine" ]` and `echo "api-key=${{ secrets[matrix.api-user] }},${{ secrets[matrix.api-key] }}" >> $GITHUB_OUTPUT`.

Locations:

- `.github/workflows/functional-tests.yml:34`

### github-env-injection (severity: high)

The 'Prepare API key' step writes values derived from `${{ secrets[matrix.api-key] }}` and `${{ secrets[matrix.api-user] }}` directly to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline embedded in the secret value could inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs.

Locations:

- `.github/workflows/functional-tests.yml:35`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tag or branch is moved or compromised. Failing references include: functional-tests.yml: `actions/checkout@v4`; linting.yml: `actions/checkout@v4`, `actions/setup-node@v4`, `actions/cache@v4`, `sibiraj-s/action-eslint@v3`, `github/codeql-action/upload-sarif@v3`, `fabasoad/reusable-workflows/.github/workflows/wf-pre-commit.yml@main`; release.yml: `fabasoad/reusable-workflows/.github/workflows/wf-github-release.yml@main`; security.yml: `fabasoad/reusable-workflows/.github/workflows/wf-security-sast.yml@main`; sync-labels.yml: `fabasoad/reusable-workflows/.github/workflows/wf-sync-labels.yml@main`; unit-tests.yml: `actions/checkout@v4`, `actions/cache@v4`, `paambaati/codeclimate-action@v6.0.0`; update-license.yml: `fabasoad/reusable-workflows/.github/workflows/wf-update-license.yml@main`.

Locations:

- `.github/workflows/functional-tests.yml:31`
- `.github/workflows/linting.yml:15`
- `.github/workflows/linting.yml:18`
- `.github/workflows/linting.yml:22`
- `.github/workflows/linting.yml:33`
- `.github/workflows/linting.yml:42`
- `.github/workflows/linting.yml:48`
- `.github/workflows/release.yml:10`
- `.github/workflows/security.yml:11`
- `.github/workflows/sync-labels.yml:10`
- `.github/workflows/unit-tests.yml:22`
- `.github/workflows/unit-tests.yml:25`
- `.github/workflows/unit-tests.yml:35`
- `.github/workflows/update-license.yml:10`

### missing-permissions (severity: medium)

Several workflow files have no top-level `permissions:` key and their jobs also lack job-level `permissions:` blocks. Without explicit permissions, workflows inherit the default repository token permissions (which may be broad). Affected files: functional-tests.yml, linting.yml, unit-tests.yml, sync-labels.yml, update-license.yml, release.yml.

Locations:

- `.github/workflows/functional-tests.yml:1`
- `.github/workflows/linting.yml:1`
- `.github/workflows/unit-tests.yml:1`
- `.github/workflows/sync-labels.yml:1`
- `.github/workflows/update-license.yml:1`
- `.github/workflows/release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 4 findings across 7 workflow files:

1. script-injection (functional-tests.yml line 34): Moved ${{ matrix.provider }}, ${{ secrets[matrix.api-user] }}, and ${{ secrets[matrix.api-key] }} from the run: shell script into the step's env: block as PROVIDER, API_USER, and API_KEY. Shell script now uses plain $PROVIDER, $API_USER, $API_KEY variables.

2. github-env-injection (functional-tests.yml line 35): Added printf '%s' "$VAR" | tr -d '\n\r' sanitization for all values written to $GITHUB_OUTPUT.

3. unpinned-uses: Pinned all 9 unique action references to full 40-char SHAs with original tag preserved in a comment: actions/checkout@v4→11d5960a, actions/setup-node@v4→49933ea5, actions/cache@v4→0057852b, sibiraj-s/action-eslint@v3→bcf41bb9, github/codeql-action/upload-sarif@v3→4187e74d, fabasoad/reusable-workflows@main→c5bd8945 (used in 5 workflow files), paambaati/codeclimate-action@v6.0.0→b74bb25d.

4. missing-permissions: Added top-level permissions: {} to functional-tests.yml, linting.yml, unit-tests.yml, sync-labels.yml, update-license.yml, and release.yml. Added job-level permissions (contents: read, security-events: write) to linting.yml eslint job and unit-tests.yml unit-tests job. Also added top-level permissions: {} to security.yml which already had job-level permissions.


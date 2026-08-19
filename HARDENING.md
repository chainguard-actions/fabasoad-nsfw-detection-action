<!-- markdownlint-disable -->

# Hardening Report: fabasoad--nsfw-detection-action/v2.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--nsfw-detection-action/v2.0.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Prepare API key' run: block in functional-tests.yml directly interpolates GitHub Actions expressions inside shell commands: `if [ "${{ matrix.provider }}" = "sightengine" ]`, `echo "api-key=${{ secrets[matrix.api-user] }},${{ secrets[matrix.api-key] }}"`, and `echo "api-key=${{ secrets[matrix.api-key] }}"`. These expressions are expanded by the template engine before the shell sees them, allowing an attacker who controls matrix values or secrets references to inject arbitrary shell commands.

Locations:

- `.github/workflows/functional-tests.yml:33`

### github-env-injection (severity: high)

The 'Prepare API key' run: block writes values derived from ${{ secrets[matrix.api-user] }}, ${{ secrets[matrix.api-key] }} directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). A newline embedded in a secret value could inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs.

Locations:

- `.github/workflows/functional-tests.yml:35`

### unpinned-uses (severity: high)

Every `uses:` reference across all workflow files uses a mutable tag instead of a full 40-character SHA commit hash, making the workflows vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Unpinned references include: actions/checkout@v4, actions/setup-node@v4, actions/cache@v4, sibiraj-s/action-eslint@v3, github/codeql-action/upload-sarif@v3, fabasoad/reusable-workflows/...@main, simbo/changes-since-last-release-action@v1, softprops/action-gh-release@v1, fischerscode/tagger@v0, github/codeql-action/init@v3, github/codeql-action/analyze@v3, anchore/scan-action@v3, micnncim/action-label-syncer@v1, paambaati/codeclimate-action@v5.0.0, FantasticFiasco/action-update-license-year@v3.

Locations:

- `.github/workflows/functional-tests.yml:31`
- `.github/workflows/linting.yml:14`
- `.github/workflows/linting.yml:17`
- `.github/workflows/linting.yml:22`
- `.github/workflows/linting.yml:37`
- `.github/workflows/linting.yml:46`
- `.github/workflows/linting.yml:50`
- `.github/workflows/release.yml:13`
- `.github/workflows/release.yml:17`
- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:30`
- `.github/workflows/security.yml:15`
- `.github/workflows/security.yml:17`
- `.github/workflows/security.yml:20`
- `.github/workflows/security.yml:23`
- `.github/workflows/security.yml:32`
- `.github/workflows/security.yml:39`
- `.github/workflows/security.yml:42`
- `.github/workflows/security.yml:47`
- `.github/workflows/sync-labels.yml:14`
- `.github/workflows/sync-labels.yml:16`
- `.github/workflows/unit-tests.yml:26`
- `.github/workflows/unit-tests.yml:29`
- `.github/workflows/unit-tests.yml:42`
- `.github/workflows/update-license.yml:11`
- `.github/workflows/update-license.yml:14`

### missing-permissions (severity: medium)

None of the 7 workflow files define a top-level `permissions:` block, and no individual job within any of these files defines a `permissions:` block either. Without explicit permissions, workflows run with the default token permissions (which may be read-write depending on repository settings), violating the principle of least privilege.

Locations:

- `.github/workflows/functional-tests.yml:1`
- `.github/workflows/linting.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/security.yml:1`
- `.github/workflows/sync-labels.yml:1`
- `.github/workflows/unit-tests.yml:1`
- `.github/workflows/update-license.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 4 findings across 7 workflow files:

1. script-injection (functional-tests.yml): Moved ${{ matrix.provider }}, ${{ secrets[matrix.api-user] }}, and ${{ secrets[matrix.api-key] }} out of the run: shell block into the step's env: block as PROVIDER, SECRET_API_USER, SECRET_API_KEY. Shell script now references plain env vars.

2. github-env-injection (functional-tests.yml): Added sanitization using `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT to prevent newline injection.

3. unpinned-uses: Pinned all 15 action references to full 40-char SHAs with original tag as comment: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 (v4), actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 (v4), actions/cache@0057852bfaa89a56745cba8c7296529d2fc39830 (v4), sibiraj-s/action-eslint@bcf41bb9abce43cdbad51ab9b3da2eddaa17eab3 (v3), github/codeql-action@b7351df727350dca84cb9d725d57dcf5bc82ba26 (v3), fabasoad/reusable-workflows@4e2279474e598bee3ae8ded28899a24bbc7bf971 (main), simbo/changes-since-last-release-action@45883b23a1c40599d6967b447b919a5663aaf23d (v1), softprops/action-gh-release@de2c0eb89ae2a093876385947365aca7b0e5f844 (v1), fischerscode/tagger@5ca3fa63ce3003fb7183cae547644b29f3b632be (v0), anchore/scan-action@3343887d815d7b07465f6fdcd395bd66508d486a (v3), micnncim/action-label-syncer@3abd5ab72fda571e69fffd97bd4e0033dd5f495c (v1), paambaati/codeclimate-action@a1831d7162ea1fbc612ffe5fb3b90278b7999d59 (v5.0.0), FantasticFiasco/action-update-license-year@f180e962fa988db222d8f03ef4636750312d1b3d (v3).

4. missing-permissions: Added `permissions: {}` at top level of all 7 workflow files. Added job-level permissions where needed: security-events: write (linting/security SARIF uploads), contents: write (release/update-license), issues: write (sync-labels), pull-requests: write (update-license).


<!-- markdownlint-disable -->

# Hardening Report: fabasoad--nsfw-detection-action/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **fabasoad--nsfw-detection-action/v3.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable tags rather than full 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved or hijacked:
- `uses: tj-actions/changed-files@v45` (line 50)
- `uses: dcarbone/install-jq-action@v3` (line 75)
These should be pinned to immutable commit SHAs, e.g. `tj-actions/changed-files@<40-char-sha> # v45`.

Locations:

- `action.yml:50`
- `action.yml:75`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate GitHub Actions expressions inside shell command strings (rule a), and also use unquoted shell variable expansions of attacker-controlled data (rule b).

**Rule (a) — direct expression interpolation in run blocks:**
Four classify steps embed `${{ steps.changed-files.outputs.added_files }}`, `${{ steps.changed-files.outputs.copied_files }}`, `${{ steps.changed-files.outputs.modified_files }}`, and `${{ steps.changed-files.outputs.renamed_files }}` directly as shell arguments. These step outputs come from an external action (tj-actions/changed-files) and contain file paths that could include shell metacharacters, enabling command injection.

Example (Classify added files step):
```
"${{ steps.changed-files.outputs.added_files }}" \
```

**Rule (b) — unquoted shell variable expansion of attacker-controlled data:**
All four classify steps use `${INPUT_PROVIDER}` unquoted in the shell path:
```
${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh \
```
`INPUT_PROVIDER` is set from `${{ inputs.provider }}` (attacker-controlled). Without double-quoting, shell metacharacters in the value can break word boundaries and enable path traversal or command injection. It should be `"${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh"`.

Locations:

- `action.yml:85`
- `action.yml:86`
- `action.yml:98`
- `action.yml:99`
- `action.yml:111`
- `action.yml:112`
- `action.yml:124`
- `action.yml:125`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed two unpinned action references by resolving their full SHA digests: tj-actions/changed-files@v45 → @48d8f15b2aaa3d255ca5af3eba4870f807ce6b3c and dcarbone/install-jq-action@v3 → @b7ef57d46ece78760b4019dbc4080a1ba2a40b45. Fixed script-injection in all four classify steps (added, copied, modified, renamed): (a) moved ${{ steps.changed-files.outputs.*_files }} expressions into env: blocks as CHANGED_FILES and referenced as "${CHANGED_FILES}" in the shell; (b) added double-quotes around the provider script path "${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh" to prevent word-splitting.


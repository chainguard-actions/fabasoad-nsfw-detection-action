<!-- markdownlint-disable -->

# Hardening Report: fabasoad--nsfw-detection-action/v3.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--nsfw-detection-action/v3.0.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (b): Unquoted shell variable expansion of untrusted data. In four `run:` blocks ('Classify added files', 'Classify copied files', 'Classify modified files', 'Classify renamed files'), the env var `${INPUT_PROVIDER}` — which holds the caller-controlled value `${{ inputs.provider }}` — is expanded **unquoted** inside the shell command: `${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh`. An unquoted expansion allows the shell to parse metacharacters (`;`, `|`, `&`, `$(...)`, path traversal `../`, etc.) out of the value before the script executes, enabling command injection. The fix is to double-quote the variable: `"${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh"`.

Locations:

- `action.yml:84`
- `action.yml:101`
- `action.yml:118`
- `action.yml:135`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansion of ${INPUT_PROVIDER} in four 'run:' blocks ('Classify added files', 'Classify copied files', 'Classify modified files', 'Classify renamed files'). In each case, the path `${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh` was wrapped in double quotes to become `"${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh"`, preventing shell word-splitting and metacharacter interpretation of the caller-controlled provider input value.


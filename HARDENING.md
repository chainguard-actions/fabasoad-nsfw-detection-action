<!-- markdownlint-disable -->

# Hardening Report: fabasoad--nsfw-detection-action/v3.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--nsfw-detection-action/v3.1.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): unquoted shell expansion of workflow-controlled data. The four 'Classify * files' steps run `${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh \`. INPUT_PROVIDER comes from inputs.provider and is expanded without quotes as part of the command path. Because the word is unquoted, the shell splits it on whitespace and expands globs. The value also chooses which script runs, so something like '../../x' would run a different file. The only validation step is skipped unless `env.SCAN_SCA_EXIT_CODE == '0'`, so it often doesn't run at all. Fix: quote the whole path, e.g. "${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh", and check INPUT_PROVIDER against an allowlist (cloudmersive|openai|picpurify|sightengine) inside the same step.

Locations:

- `action.yml:88`
- `action.yml:107`
- `action.yml:126`
- `action.yml:145`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

In all four 'Classify * files' steps, the provider script path is now quoted, so it can't be split on spaces or glob-expanded. Each step also checks INPUT_PROVIDER against an allowlist (cloudmersive|openai|picpurify|sightengine) with a POSIX case statement and stops with an error for any other value. The allowlist matches the scripts in src/providers. The steps still use shell: sh. I didn't change anything else.


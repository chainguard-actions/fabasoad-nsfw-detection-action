<!-- markdownlint-disable -->

# Hardening Report: fabasoad--nsfw-detection-action/v3.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--nsfw-detection-action/v3.0.2** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): in the 'Classify added files' step, the workflow-controlled env var INPUT_PROVIDER (from inputs.provider) is expanded without quotes in the command path: `${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh \`. The shell will split it on whitespace and expand globs. Quote it: "${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh".

Locations:

- `action.yml:86`

### script-injection (severity: high)

Rule (b): in the 'Classify copied files' step, INPUT_PROVIDER (from inputs.provider) is expanded without quotes: `${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh \`.

Locations:

- `action.yml:103`

### script-injection (severity: high)

Rule (b): in the 'Classify modified files' step, INPUT_PROVIDER (from inputs.provider) is expanded without quotes: `${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh \`.

Locations:

- `action.yml:120`

### script-injection (severity: high)

Rule (b): in the 'Classify renamed files' step, INPUT_PROVIDER (from inputs.provider) is expanded without quotes: `${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh \`.

Locations:

- `action.yml:137`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index ff74e2a..aee2d26 100644
--- a/action.yml
+++ b/action.yml
@@ -1,4 +1,3 @@
----
 name: NSFW Detection Action
 author: Yevhen Fabizhevskyi
 description: This GitHub Action detects NSFW content in committed files.
@@ -28,6 +27,23 @@ inputs:
 runs:
   using: composite
   steps:
     - name: Validate inputs
       if: env.SCAN_SCA_EXIT_CODE == '0'
       id: validate-inputs
@@ -46,125 +62,72 @@ runs:
           "${INPUT_TYPES}"
       shell: sh
       working-directory: ${{ github.action_path }}/src
-
     - name: Get changed files
       id: changed-files
       uses: tj-actions/changed-files@9426d40962ed5378910ee2e21d5f8c6fcbf2dd96 # v47.0.6
       with:
         files: "**.{${{ inputs.extensions }}}"
-
     - name: Setup jq
-      if: (
-        (
-        contains(format(',{0},', inputs.types), ',added,')
-        && steps.changed-files.outputs.added_files_count != '0'
-        )
-        || (
-        contains(format(',{0},', inputs.types), ',copied,')
-        && steps.changed-files.outputs.copied_files_count != '0'
-        )
-        || (
-        contains(format(',{0},', inputs.types), ',modified,')
-        && steps.changed-files.outputs.modified_files_count != '0'
-        )
-        || (
-        contains(format(',{0},', inputs.types), ',renamed,')
-        && steps.changed-files.outputs.renamed_files_count != '0'
-        )
-        )
       uses: dcarbone/install-jq-action@4fcb5062d7ce9bc4382d1a352d19ba3ba2c317c1 # v4.0.1
-
+      if: ( ( contains(format(',{0},', inputs.types), ',added,') && steps.changed-files.outputs.added_files_count != '0' ) || ( contains(format(',{0},', inputs.types), ',copied,') && steps.changed-files.outputs.copied_files_count != '0' ) || ( contains(format(',{0},', inputs.types), ',modified,') && steps.changed-files.outputs.modified_files_count != '0' ) || ( contains(format(',{0},', inputs.types), ',renamed,') && steps.changed-files.outputs.renamed_files_count != '0' ) )
     - name: Classify added files
-      if: (
-        contains(format(',{0},', inputs.types), ',added,')
-        && steps.changed-files.outputs.added_files_count != '0'
-        )
+      if: ( contains(format(',{0},', inputs.types), ',added,') && steps.changed-files.outputs.added_files_count != '0' )
       env:
         INPUT_PROVIDER: "${{ inputs.provider }}"
         INPUT_API_KEY: "${{ inputs.api-key }}"
         INPUT_THRESHOLD: "${{ inputs.threshold }}"
         STEP_CHANGED_FILES_OUTPUT_ADDED_FILES: "${{ steps.changed-files.outputs.added_files }}"
       run: |
-        ${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh \
+        "${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh" \
           "${STEP_CHANGED_FILES_OUTPUT_ADDED_FILES}" \
           "created" \
           "${INPUT_API_KEY}" \
           "${INPUT_THRESHOLD}"
       shell: sh
-
     - name: Classify copied files
-      if: (
-        contains(format(',{0},', inputs.types), ',copied,')
-        && steps.changed-files.outputs.copied_files_count != '0'
-        )
+      if: ( contains(format(',{0},', inputs.types), ',copied,') && steps.changed-files.outputs.copied_files_count != '0' )
       env:
         INPUT_PROVIDER: "${{ inputs.provider }}"
         INPUT_API_KEY: "${{ inputs.api-key }}"
         INPUT_THRESHOLD: "${{ inputs.threshold }}"
         STEP_CHANGED_FILES_OUTPUT_COPIED_FILES: "${{ steps.changed-files.outputs.copied_files }}"
       run: |
-        ${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh \
+        "${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh" \
           "${STEP_CHANGED_FILES_OUTPUT_COPIED_FILES}" \
           "copied" \
           "${INPUT_API_KEY}" \
           "${INPUT_THRESHOLD}"
       shell: sh
-
     - name: Classify modified files
-      if: (
-        contains(format(',{0},', inputs.types), ',modified,')
-        && steps.changed-files.outputs.modified_files_count != '0'
-        )
+      if: ( contains(format(',{0},', inputs.types), ',modified,') && steps.changed-files.outputs.modified_files_count != '0' )
       env:
         INPUT_PROVID
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, prior-fix

**Notes:**

Put double quotes around the provider script path "${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh" in all four Classify steps in action.yml. The value is now one argument, so whitespace in it no longer splits it into several words and glob characters are no longer expanded. This redoes the security part of the previous harden's patch. The rest of that patch reflowed the multi-line if: expressions onto one line and removed blank lines, which doesn't change behaviour. That diff was cut off: it also added about 17 lines near the start of runs.steps whose content I couldn't see, so I didn't restore them. Every other ${{ }} expression was already passed through env:, and the uses: references were already pinned to commit SHAs.


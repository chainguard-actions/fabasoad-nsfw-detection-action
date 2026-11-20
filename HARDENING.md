<!-- markdownlint-disable -->

# Hardening Report: fabasoad--nsfw-detection-action/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--nsfw-detection-action/v3.0.1** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references actions by mutable tags instead of 40-char commit SHAs: `uses: tj-actions/changed-files@v45` (line 51; this action was compromised in a supply-chain attack in 2025) and `uses: dcarbone/install-jq-action@v3` (line 74).

Locations:

- `action.yml:51`
- `action.yml:74`

### script-injection (severity: high)

Rule (a): 'Classify added files' step interpolates an expression directly in run: `"${{ steps.changed-files.outputs.added_files }}" \` (file names come from the PR and an attacker controls them, e.g. a file named with $(...) or a quote). Rule (b): `${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh \` expands the workflow-controlled INPUT_PROVIDER and GITHUB_ACTION_PATH without quotes.

Locations:

- `action.yml:86`
- `action.yml:87`

### script-injection (severity: high)

Rule (a): 'Classify copied files' step interpolates `"${{ steps.changed-files.outputs.copied_files }}" \` directly in run:. Rule (b): `${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh` is not quoted.

Locations:

- `action.yml:103`
- `action.yml:104`

### script-injection (severity: high)

Rule (a): 'Classify modified files' step interpolates `"${{ steps.changed-files.outputs.modified_files }}" \` directly in run:. Rule (b): `${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh` is not quoted.

Locations:

- `action.yml:120`
- `action.yml:121`

### script-injection (severity: high)

Rule (a): 'Classify renamed files' step interpolates `"${{ steps.changed-files.outputs.renamed_files }}" \` directly in run:. Rule (b): `${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh` is not quoted.

Locations:

- `action.yml:137`
- `action.yml:138`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 0ed71f1..fa802cc 100644
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
@@ -46,121 +62,72 @@ runs:
           "${INPUT_TYPES}"
       shell: sh
       working-directory: ${{ github.action_path }}/src
-
     - name: Get changed files
       id: changed-files
-      uses: tj-actions/changed-files@v45
+      uses: tj-actions/changed-files@48d8f15b2aaa3d255ca5af3eba4870f807ce6b3c # v45
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
-      uses: dcarbone/install-jq-action@v3
-
+      if: ( ( contains(format(',{0},', inputs.types), ',added,') && steps.changed-files.outputs.added_files_count != '0' ) || ( contains(format(',{0},', inputs.types), ',copied,') && steps.changed-files.outputs.copied_files_count != '0' ) || ( contains(format(',{0},', inputs.types), ',modified,') && steps.changed-files.outputs.modified_files_count != '0' ) || ( contains(format(',{0},', inputs.types), ',renamed,') && steps.changed-files.outputs.renamed_files_count != '0' ) )
+      uses: dcarbone/install-jq-action@b7ef57d46ece78760b4019dbc4080a1ba2a40b45 # v3
     - name: Classify added files
-      if: (
-        contains(format(',{0},', inputs.types), ',added,')
-        && steps.changed-files.outputs.added_files_count != '0'
-        )
+      if: ( contains(format(',{0},', inputs.types), ',added,') && steps.changed-files.outputs.added_files_count != '0' )
       env:
         INPUT_PROVIDER: ${{ inputs.provider }}
         INPUT_API_KEY: ${{ inputs.api-key }}
         INPUT_THRESHOLD: ${{ inputs.threshold }}
+        CHANGED_FILES: ${{ steps.changed-files.outputs.added_files }}
       run: |
-        ${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh \
-          "${{ steps.changed-files.outputs.added_files }}" \
+        "${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh" \
+          "${CHANGED_FILES}" \
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
         INPUT_PROVIDER: ${{ inputs.provider }}
         INPUT_API_KEY: ${{ inputs.api-key }}
         INPUT_THRESHOLD: ${{ inputs.threshold }}
+        CHANGED_FILES: ${{ steps.changed-files.outputs.copied_files }}
       run: |
-        ${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh \
-          "${{ steps.changed-files.outputs.copied_files }}" \
+        "${GITHUB_ACTION_PATH}/src/providers/${INPUT_PROVIDER}.sh" \
+          "${CHANGED_FILES}" \
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
+      if: ( contains(format(',{0},', inputs.types), ',modified,') && ste
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, prior-fix

**Notes:**

I pinned tj-actions/changed-files to 48d8f15b2aaa3d255ca5af3eba4870f807ce6b3c (v45) and dcarbone/install-jq-action to b7ef57d46ece78760b4019dbc4080a1ba2a40b45 (v3). Both SHAs came from lookup_action_sha and are the same ones the previous harden used. In each of the four 'Classify * files' steps, the changed-files output now goes through a CHANGED_FILES env var instead of being written straight into run:, and the provider script path is in double quotes. I didn't test any of this. The previous patch also added a 17-line block at the start of steps:, but its text is cut off in the finding, so I couldn't re-apply it. Its other changes are back in place except one: it also flattened the multi-line if: conditions, which made no security difference, so I left them as upstream wrote them.


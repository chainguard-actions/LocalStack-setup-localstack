# Hardening Report: LocalStack--setup-localstack/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **LocalStack--setup-localstack/v0.3.0** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct interpolation of `${{ github.event.number }}` in a `run:` shell command. The value is not first assigned to an environment variable, allowing script injection if the event number contains unexpected content.

Locations:

- `prepare/action.yml:15`

### script-injection (severity: high)

Direct interpolation of `${{ inputs.ci-project }}` in a `run:` shell command (`export CI_PROJECT=${{ inputs.ci-project }}`). Attacker-controlled input is embedded directly in the shell script without routing through an env var, enabling script injection.

Locations:

- `startup/action.yml:57`

### script-injection (severity: high)

Direct interpolation of `${{ inputs.preview-cmd }}` as the entire body of a `run:` block. This allows an attacker to supply arbitrary shell commands via the `preview-cmd` input, resulting in full remote code execution on the runner. Additionally, `${{ inputs.localstack-api-key }}`, `${{ inputs.auto-load-pod }}`, `${{ inputs.extension-auto-install }}`, and `${{ inputs.lifetime }}` are all interpolated directly into other `run:` blocks in this file without env-var indirection.

Locations:

- `ephemeral/startup/action.yml:130`
- `ephemeral/startup/action.yml:79`
- `ephemeral/startup/action.yml:83`
- `ephemeral/startup/action.yml:84`
- `ephemeral/startup/action.yml:85`

### script-injection (severity: high)

Direct interpolation of `${{ inputs.localstack-api-key }}` inside a `run:` shell block (`AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"`). The input value is embedded directly in the shell script without env-var indirection, enabling injection.

Locations:

- `ephemeral/shutdown/action.yml:36`

### script-injection (severity: high)

Direct interpolation of `${{ inputs.preview-url }}` inside a `run:` shell block. The input value is embedded directly in the shell script and also written to `$GITHUB_ENV` without sanitization, enabling both script injection and environment injection.

Locations:

- `finish/action.yml:52`

### github-env-injection (severity: high)

The `run:` block in the 'Load the Ephemeral Instance URL' step writes `${{ inputs.preview-url }}` directly to `$GITHUB_ENV` (via `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker can inject arbitrary environment variables by supplying a newline in the `preview-url` input.

Locations:

- `finish/action.yml:53`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script injection and github-env-injection findings across 5 files:

1. **prepare/action.yml** (line 15): Moved `${{ github.event.number }}` to `PR_NUMBER` env var; run now uses `echo "$PR_NUMBER" > ./pr-id.txt`.

2. **startup/action.yml** (line 57): Moved `${{ inputs.ci-project }}` to `CI_PROJECT_INPUT` env var; run now uses `export CI_PROJECT="${CI_PROJECT_INPUT}"`.

3. **ephemeral/startup/action.yml** (lines 79, 83, 84, 85, 130):
   - In 'Create preview environment' step: Added env block with `INPUT_LOCALSTACK_API_KEY`, `INPUT_AUTO_LOAD_POD`, `INPUT_EXTENSION_AUTO_INSTALL`, `INPUT_LIFETIME`; replaced all direct `${{ inputs.* }}` interpolations with env var references.
   - In 'Run preview deployment' step: Moved `${{ inputs.preview-cmd }}` to `PREVIEW_CMD` env var; run now uses `eval "$PREVIEW_CMD"`.
   - In 'Print logs of ephemeral instance' step: Added `INPUT_LOCALSTACK_API_KEY` env var; replaced direct `${{ inputs.localstack-api-key }}` interpolation.

4. **ephemeral/shutdown/action.yml** (line 36): Added `INPUT_LOCALSTACK_API_KEY` env var to 'Shutdown ephemeral instance' step; replaced direct `${{ inputs.localstack-api-key }}` interpolation.

5. **finish/action.yml** (lines 52-53): Added `INPUT_PREVIEW_URL` env var to 'Load the Ephemeral Instance URL' step; replaced both direct `${{ inputs.preview-url }}` interpolations; added `printf '%s' ... | tr -d '\n\r'` sanitization before all writes to `$GITHUB_ENV` to prevent environment injection.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all three instances of `${{ github.action_path }}` being interpolated directly in `run:` shell strings:
1. `ephemeral/shutdown/action.yml` (line 44): Added `ACTION_PATH: ${{ github.action_path }}` to the `env:` block of the 'Shutdown ephemeral instance' step and changed `source ${{ github.action_path }}/../retry-function.sh` to `source "$ACTION_PATH/../retry-function.sh"`.
2. `ephemeral/startup/action.yml` (line 71): Added `ACTION_PATH: ${{ github.action_path }}` to the `env:` block of the 'Create preview environment' step and updated the source call accordingly.
3. `ephemeral/startup/action.yml` (line 165): Added `ACTION_PATH: ${{ github.action_path }}` to the `env:` block of the 'Print logs of ephemeral instance' step and updated the source call accordingly.


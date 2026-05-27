# Hardening Report: LocalStack--setup-localstack/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **LocalStack--setup-localstack/v0.3.2** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct interpolation of `${{ github.event.number }}` in a `run:` shell command. The value is not first assigned to an env: variable, allowing an attacker to inject arbitrary shell commands via a crafted event payload.

Locations:

- `prepare/action.yml:14`

### script-injection (severity: high)

Multiple `inputs.*` expressions are directly interpolated inside `run:` shell commands in the 'Create preview environment' step: `${{ inputs.localstack-api-key }}`, `${{ inputs.auto-load-pod }}`, `${{ inputs.extension-auto-install }}`, and `${{ inputs.lifetime }}` are all embedded directly in shell strings without going through env: variables. An attacker controlling these inputs can inject arbitrary shell commands.

Locations:

- `ephemeral/startup/action.yml:55`
- `ephemeral/startup/action.yml:75`
- `ephemeral/startup/action.yml:76`
- `ephemeral/startup/action.yml:77`

### script-injection (severity: high)

The 'Run preview deployment' step directly executes `${{ inputs.preview-cmd }}` as a shell command in a `run:` block. This is a critical script injection: any attacker-controlled value for `inputs.preview-cmd` is executed verbatim as a shell command with no sanitization or env: indirection.

Locations:

- `ephemeral/startup/action.yml:148`

### script-injection (severity: high)

The 'Print logs of ephemeral instance' step directly interpolates `${{ inputs.localstack-api-key }}` inside a `run:` shell command (`AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"`). This bypasses env: variable indirection and allows shell injection.

Locations:

- `ephemeral/startup/action.yml:163`

### script-injection (severity: high)

The 'Shutdown ephemeral instance' step directly interpolates `${{ inputs.localstack-api-key }}` inside a `run:` shell command (`AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"`). An attacker-controlled input value is embedded directly in the shell string.

Locations:

- `ephemeral/shutdown/action.yml:35`

### script-injection (severity: high)

The 'Start LocalStack' step in startup/action.yml directly interpolates `${{ inputs.ci-project }}` into a `run:` shell command (`export CI_PROJECT=${{ inputs.ci-project }}`). Although most inputs are correctly passed via env:, `inputs.ci-project` is interpolated directly, allowing shell injection.

Locations:

- `startup/action.yml:65`

### github-env-injection (severity: high)

The 'Load the Ephemeral Instance URL' step in finish/action.yml writes `inputs.preview-url` directly to `$GITHUB_ENV` without sanitization: `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. An attacker-controlled `inputs.preview-url` value containing newlines could inject arbitrary environment variables (e.g. `key=value\nATTACKER_VAR=malicious`) into the runner environment. The required `printf '%s' ... | tr -d '\n\r'` sanitization step is absent.

Locations:

- `finish/action.yml:60`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 7 security findings across 5 files:

1. prepare/action.yml: Moved `${{ github.event.number }}` to env var `PR_NUMBER` to prevent script injection.

2. ephemeral/startup/action.yml (Create preview environment): Moved `inputs.localstack-api-key`, `inputs.auto-load-pod`, `inputs.extension-auto-install`, `inputs.lifetime`, and `github.action_path` to env: block variables to prevent script injection.

3. ephemeral/startup/action.yml (Run preview deployment): Moved `inputs.preview-cmd` to env var `PREVIEW_CMD` and used `eval "$PREVIEW_CMD"` to execute it safely.

4. ephemeral/startup/action.yml (Print logs): Moved `inputs.localstack-api-key` and `github.action_path` to env: block variables.

5. ephemeral/shutdown/action.yml: Moved `inputs.localstack-api-key` and `github.action_path` to env: block variables (`INPUT_LOCALSTACK_API_KEY`, `ACTION_PATH`).

6. startup/action.yml: Added `CI_PROJECT_INPUT: ${{ inputs.ci-project }}` to env: block and replaced direct interpolation with `"${CI_PROJECT_INPUT}"` reference.

7. finish/action.yml: Moved `inputs.preview-url` to env var `INPUT_PREVIEW_URL` and added `printf '%s' ... | tr -d '\n\r'` sanitization before writing to $GITHUB_ENV to prevent newline injection.


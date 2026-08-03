<!-- markdownlint-disable -->

# Hardening Report: tj-actions--docker-run/v2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--docker-run/v2.2.0** was hardened automatically. 9 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates multiple `inputs.*` expressions into shell commands without quoting or sanitization. Specifically: `${{ inputs.name }}`, `${{ inputs.options }}`, `${{ inputs.image }}`, and `${{ inputs.args }}` are all expanded inline in the `docker run` command (sub-rule a). An attacker-controlled caller can supply values containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) to achieve arbitrary command execution on the runner. The offending line is: `docker run -v "/var/run/docker.sock":"/var/run/docker.sock" --name ${{ inputs.name }} ${{ inputs.options }} ${{ inputs.image }} ${{ inputs.args }}`

Locations:

- `action.yml:23`

### github-env-injection (severity: high)

The `run:` block in action.yml writes a value derived from the untrusted input `${{ inputs.name }}` directly into `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline injected via `inputs.name` could allow an attacker to inject arbitrary key=value pairs into the GitHub output environment. The offending line is: `echo "container_id=$(docker ps -a --filter name=${{ inputs.name }} -q)" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:24`

### unpinned-uses (severity: high)

All `uses:` references across every workflow file use mutable tag or version strings instead of immutable 40-character SHA commit hashes. This exposes the action to supply-chain attacks where a compromised or malicious tag update could execute arbitrary code in CI. Unpinned references found:
- `hmarr/auto-approve-action@v3`
- `pascalgn/automerge-action@v0.15.5`
- `actions/first-interaction@v1`
- `actions/checkout@v3` (multiple files)
- `cirrus-actions/rebase@1.8`
- `tj-actions/release-tagger@v2`
- `tj-actions/sync-release-version@v13`
- `tj-actions/github-changelog-generator@v1.17`
- `peter-evans/create-pull-request@v4` (multiple files)
- `reviewdog/action-shellcheck@v1`
- `docker/setup-qemu-action@v2.1.0`
- `docker/setup-buildx-action@v2.2.1`
- `docker/build-push-action@v3`
- `tj-actions/auto-doc@v1`
- `tj-actions/remark@v3`
- `tj-actions/verify-changed-files@v13`

Locations:

- `.github/workflows/auto-approve.yml:8`
- `.github/workflows/auto-merge.yml:16`
- `.github/workflows/greetings.yml:7`
- `.github/workflows/rebase.yml:9`
- `.github/workflows/rebase.yml:13`
- `.github/workflows/sync-release-version.yml:9`
- `.github/workflows/sync-release-version.yml:11`
- `.github/workflows/sync-release-version.yml:14`
- `.github/workflows/sync-release-version.yml:18`
- `.github/workflows/sync-release-version.yml:22`
- `.github/workflows/test.yml:13`
- `.github/workflows/test.yml:16`
- `.github/workflows/test.yml:19`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:26`
- `.github/workflows/update-readme.yml:8`
- `.github/workflows/update-readme.yml:11`
- `.github/workflows/update-readme.yml:14`
- `.github/workflows/update-readme.yml:17`
- `.github/workflows/update-readme.yml:30`

### missing-permissions (severity: medium)

None of the 7 workflow files define a top-level `permissions:` block, and no individual job within any of these files defines a `permissions:` block either. Without explicit permissions, workflows run with the repository's default token permissions, which are often overly broad (e.g., write access to contents, pull-requests, etc.). Each workflow file should declare the minimal permissions required.

Locations:

- `.github/workflows/auto-approve.yml:1`
- `.github/workflows/auto-merge.yml:1`
- `.github/workflows/greetings.yml:1`
- `.github/workflows/rebase.yml:1`
- `.github/workflows/sync-release-version.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-readme.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.name }}" appears directly in run: block of step "Docker Run"; move to env: map

Locations:

- `action.yml:31`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.options }}" appears directly in run: block of step "Docker Run"; move to env: map

Locations:

- `action.yml:31`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.image }}" appears directly in run: block of step "Docker Run"; move to env: map

Locations:

- `action.yml:31`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.args }}" appears directly in run: block of step "Docker Run"; move to env: map

Locations:

- `action.yml:31`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.name }}" appears directly in run: block of step "Docker Run"; move to env: map

Locations:

- `action.yml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all findings:

1. action.yml script-injection/static-inline-injection: Moved all ${{ inputs.* }} expressions to env: block (INPUT_NAME, INPUT_IMAGE, INPUT_OPTIONS, INPUT_ARGS). Used quoted variables in docker run command. INPUT_OPTIONS and INPUT_ARGS are tokenized into bash arrays using xargs-based parsing to safely handle whitespace-separated list arguments.

2. action.yml github-env-injection: Sanitized INPUT_NAME with `printf '%s' "$INPUT_NAME" | tr -d '\n\r'` before writing container_id to $GITHUB_OUTPUT.

3. unpinned-uses: Pinned all 16 action references across 7 workflow files to full 40-char commit SHAs with original tag as comment: hmarr/auto-approve-action@7d0ab8f, pascalgn/automerge-action@eb68b06, actions/first-interaction@2ec0f0f, actions/checkout@a37ce91, cirrus-actions/rebase@b87d481, tj-actions/release-tagger@26b7644, tj-actions/sync-release-version@2a7ef0d, tj-actions/github-changelog-generator@8fd0db0, peter-evans/create-pull-request@38e0b6e, reviewdog/action-shellcheck@4c07458, docker/setup-qemu-action@e81a89b, docker/setup-buildx-action@8c0edbc, docker/build-push-action@1104d47, tj-actions/auto-doc@e7f517d, tj-actions/remark@10fc407, tj-actions/verify-changed-files@f557547.

4. missing-permissions: Added minimal top-level permissions blocks to all 7 workflow files.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/test.yml at the 'Show output' step. Moved `${{ toJSON(steps.test-app-1.outputs) }}` and `${{ toJSON(steps.test-app-2.outputs) }}` from direct shell interpolation into the step's `env:` block as `TEST_APP_1_OUTPUTS` and `TEST_APP_2_OUTPUTS` respectively. The shell script now references these via `"$TEST_APP_1_OUTPUTS"` and `"$TEST_APP_2_OUTPUTS"` with proper double-quoting, eliminating the injection risk.


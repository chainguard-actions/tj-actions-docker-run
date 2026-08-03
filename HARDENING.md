<!-- markdownlint-disable -->

# Hardening Report: tj-actions--docker-run/v2.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--docker-run/v2.2.1** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

action.yml run: block directly interpolates ${{ inputs.name }}, ${{ inputs.options }}, ${{ inputs.image }}, and ${{ inputs.args }} into shell commands. These are attacker-controlled values that flow through YAML template substitution before the shell sees them, enabling command injection. Sub-rule (a): direct expression interpolation in run: shell string. Offending line: `docker run -v "/var/run/docker.sock":"/var/run/docker.sock" --name ${{ inputs.name }} ${{ inputs.options }} ${{ inputs.image }} ${{ inputs.args }}`

Locations:

- `action.yml:30`

### github-env-injection (severity: high)

action.yml run: block writes ${{ inputs.name }} directly to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). An attacker-controlled input containing newlines can inject arbitrary key=value pairs into the GitHub output environment. Offending line: `echo "container_id=$(docker ps -a --filter name=${{ inputs.name }} -q)" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:31`

### script-injection (severity: high)

.github/workflows/test.yml run: block directly interpolates ${{ toJSON(steps.test-app-1.outputs) }} and ${{ toJSON(steps.test-app-2.outputs) }} into shell commands. Sub-rule (a): any ${{ ... }} expression inside a run: shell string is a script-injection risk regardless of context. Offending lines: `echo '${{ toJSON(steps.test-app-1.outputs) }}'` and `echo '${{ toJSON(steps.test-app-2.outputs) }}'`

Locations:

- `.github/workflows/test.yml:50`

### unpinned-uses (severity: high)

All uses: references across workflow files use mutable tag/version refs instead of pinned 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Unpinned references found:
- auto-approve.yml: hmarr/auto-approve-action@v3
- auto-merge.yml: pascalgn/automerge-action@v0.15.6
- greetings.yml: actions/first-interaction@v1
- rebase.yml: actions/checkout@v3, cirrus-actions/rebase@1.8
- sync-release-version.yml: actions/checkout@v3, tj-actions/release-tagger@v3, tj-actions/sync-release-version@v13, tj-actions/git-cliff@v1, peter-evans/create-pull-request@v5
- test.yml: actions/checkout@v3, reviewdog/action-shellcheck@v1, docker/setup-qemu-action@v2.2.0, docker/setup-buildx-action@v2.8.0, docker/build-push-action@v4
- update-readme.yml: actions/checkout@v3, tj-actions/auto-doc@v2, tj-actions/remark@v3, tj-actions/verify-changed-files@v16, peter-evans/create-pull-request@v5

Locations:

- `.github/workflows/auto-approve.yml:8`
- `.github/workflows/auto-merge.yml:16`
- `.github/workflows/greetings.yml:7`
- `.github/workflows/rebase.yml:10`
- `.github/workflows/rebase.yml:13`
- `.github/workflows/sync-release-version.yml:9`
- `.github/workflows/sync-release-version.yml:11`
- `.github/workflows/sync-release-version.yml:13`
- `.github/workflows/sync-release-version.yml:15`
- `.github/workflows/sync-release-version.yml:17`
- `.github/workflows/test.yml:16`
- `.github/workflows/test.yml:19`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:25`
- `.github/workflows/test.yml:28`
- `.github/workflows/update-readme.yml:9`
- `.github/workflows/update-readme.yml:12`
- `.github/workflows/update-readme.yml:14`
- `.github/workflows/update-readme.yml:16`
- `.github/workflows/update-readme.yml:22`

### missing-permissions (severity: medium)

None of the 7 workflow files define a top-level permissions: block, and none of the individual jobs define job-level permissions: blocks either. This means all jobs run with the default (potentially write-all) token permissions, violating least-privilege. Affected files: auto-approve.yml, auto-merge.yml, greetings.yml, rebase.yml, sync-release-version.yml, test.yml, update-readme.yml.

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

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings across action.yml and 7 workflow files:

1. action.yml script-injection/static-inline-injection: Moved all ${{ inputs.* }} expressions to env: block. Used xargs-based array tokenization for list-type inputs (options, args) and quoted env vars for single-value inputs (name, image).

2. action.yml github-env-injection: Added `printf '%s' "$INPUT_NAME" | tr -d '\n\r'` sanitization before writing container_id to $GITHUB_OUTPUT.

3. test.yml script-injection: Moved toJSON expressions to env: vars (TEST_APP_1_OUTPUTS, TEST_APP_2_OUTPUTS) and referenced them as plain env vars in the run: block.

4. unpinned-uses: Pinned all action references to full 40-char commit SHAs with tag comments: hmarr/auto-approve-action@7d0ab8f, pascalgn/automerge-action@22948e0, actions/first-interaction@2ec0f0f, actions/checkout@a37ce91, cirrus-actions/rebase@b87d481, tj-actions/release-tagger@6bb53bc, tj-actions/sync-release-version@2a7ef0d, tj-actions/git-cliff@75599f7, peter-evans/create-pull-request@4e1beaa, reviewdog/action-shellcheck@4c07458, docker/setup-qemu-action@2b82ce8, docker/setup-buildx-action@16c0bc4, docker/build-push-action@0a97817, tj-actions/auto-doc@a84f608, tj-actions/remark@10fc407, tj-actions/verify-changed-files@54e20d3.

5. missing-permissions: Added least-privilege permissions blocks to all 7 workflow files.


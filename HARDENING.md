<!-- markdownlint-disable -->

# Hardening Report: tj-actions--docker-run/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--docker-run/v2.0.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): action.yml line 23 directly interpolates three untrusted inputs — ${{ inputs.options }}, ${{ inputs.image }}, and ${{ inputs.args }} — inside a run: shell command string. Any caller of this composite action can supply values containing shell metacharacters (semicolons, backticks, $(...), etc.) that will be executed by bash before the shell ever sees them as data. The offending line is: `docker run --rm -v "/var/run/docker.sock":"/var/run/docker.sock" ${{ inputs.options }} ${{ inputs.image }} ${{ inputs.args }}`

Locations:

- `action.yml:23`

### unpinned-uses (severity: high)

Every uses: reference across all workflow files uses a mutable tag or version string instead of a pinned 40-character commit SHA, making the action vulnerable to supply-chain attacks if any upstream action is compromised or its tag is moved. Failing references include: auto-approve.yml: hmarr/auto-approve-action@v3; auto-merge.yml: pascalgn/automerge-action@v0.15.5; rebase.yml: actions/checkout@v3, cirrus-actions/rebase@1.8; sync-release-version.yml: actions/checkout@v3, tj-actions/release-tagger@v2, tj-actions/sync-release-version@v13, tj-actions/github-changelog-generator@v1.17, peter-evans/create-pull-request@v4; test.yml: actions/checkout@v3, reviewdog/action-shellcheck@v1, docker/setup-qemu-action@v2.1.0, docker/setup-buildx-action@v2.2.1, docker/build-push-action@v3; update-readme.yml: actions/checkout@v3.2.0, tj-actions/auto-doc@v1.7.2, tj-actions/remark@v3, tj-actions/verify-changed-files@v13, peter-evans/create-pull-request@v4.2.3.

Locations:

- `.github/workflows/auto-approve.yml:10`
- `.github/workflows/auto-merge.yml:19`
- `.github/workflows/rebase.yml:11`
- `.github/workflows/rebase.yml:14`
- `.github/workflows/sync-release-version.yml:9`
- `.github/workflows/sync-release-version.yml:12`
- `.github/workflows/sync-release-version.yml:15`
- `.github/workflows/sync-release-version.yml:20`
- `.github/workflows/sync-release-version.yml:25`
- `.github/workflows/test.yml:14`
- `.github/workflows/test.yml:17`
- `.github/workflows/test.yml:20`
- `.github/workflows/test.yml:24`
- `.github/workflows/test.yml:28`
- `.github/workflows/update-readme.yml:11`
- `.github/workflows/update-readme.yml:15`
- `.github/workflows/update-readme.yml:18`
- `.github/workflows/update-readme.yml:21`
- `.github/workflows/update-readme.yml:33`

### missing-permissions (severity: medium)

None of the 6 workflow files define a top-level permissions: key, and none of their jobs define a job-level permissions: key. Without explicit permissions, workflows inherit the repository's default token permissions (often write-all), violating the principle of least privilege. Affected files: auto-approve.yml, auto-merge.yml, rebase.yml, sync-release-version.yml, test.yml, update-readme.yml.

Locations:

- `.github/workflows/auto-approve.yml:1`
- `.github/workflows/auto-merge.yml:1`
- `.github/workflows/rebase.yml:1`
- `.github/workflows/sync-release-version.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-readme.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.options }}" appears directly in run: block of step "Docker Run"; move to env: map

Locations:

- `action.yml:23`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.image }}" appears directly in run: block of step "Docker Run"; move to env: map

Locations:

- `action.yml:23`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.args }}" appears directly in run: block of step "Docker Run"; move to env: map

Locations:

- `action.yml:23`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings: (1) Script injection in action.yml: moved inputs.options, inputs.image, and inputs.args into env: block; used xargs-based array tokenization for list inputs (options, args) and direct quoted expansion for the single-value input (image). (2) Pinned all 19 unpinned action references across 6 workflow files to full 40-char commit SHAs with original tag preserved in comments. (3) Added minimal top-level permissions blocks to all 6 workflow files: test.yml gets contents:read only; auto-approve gets pull-requests:write; the rest get contents:write and pull-requests:write as needed for their merge/PR-creation operations.


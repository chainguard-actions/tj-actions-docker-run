<!-- markdownlint-disable -->

# Hardening Report: tj-actions--docker-run/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--docker-run/v2.1.0** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

action.yml run: block directly interpolates ${{ inputs.name }}, ${{ inputs.options }}, ${{ inputs.image }}, and ${{ inputs.args }} into shell commands without quoting or env-var indirection. An attacker-controlled input value containing shell metacharacters (;, |, &, $(...), etc.) will be executed by the shell. Rule (a): direct expression interpolation in a run: block. Offending lines:
  docker run ... --name ${{ inputs.name }} ${{ inputs.options }} ${{ inputs.image }} ${{ inputs.args }}
  echo "container_id=$(docker ps -a --filter name=${{ inputs.name }} -q)" >> "$GITHUB_OUTPUT"

Locations:

- `action.yml:30`
- `action.yml:31`

### github-env-injection (severity: high)

action.yml run: block writes ${{ inputs.name }} directly into $GITHUB_OUTPUT without sanitization. The value is interpolated into the shell command `echo "container_id=$(docker ps -a --filter name=${{ inputs.name }} -q)" >> "$GITHUB_OUTPUT"`. A newline character in inputs.name would allow injecting arbitrary key=value pairs into GITHUB_OUTPUT. The required sanitization step (printf '%s' "$VAR" | tr -d '\n\r') is absent.

Locations:

- `action.yml:31`

### script-injection (severity: high)

test.yml run: block directly interpolates ${{ toJSON(steps.test-app-1.outputs) }} and ${{ toJSON(steps.test-app-2.outputs) }} into shell echo commands. Rule (a): any ${{ ... }} expression inside a run: block is a script-injection risk because YAML template substitution occurs before the shell parses the string. Offending lines:
  echo '${{ toJSON(steps.test-app-1.outputs) }}'
  echo '${{ toJSON(steps.test-app-2.outputs) }}'

Locations:

- `.github/workflows/test.yml:51`
- `.github/workflows/test.yml:52`

### missing-permissions (severity: medium)

None of the 6 workflow files define a top-level `permissions:` key, and no individual job within any of these files defines a `permissions:` key. Without explicit permissions, workflows run with the default (potentially broad) GITHUB_TOKEN permissions. All 6 workflow files are affected: auto-approve.yml, auto-merge.yml, rebase.yml, sync-release-version.yml, test.yml, update-readme.yml.

Locations:

- `.github/workflows/auto-approve.yml:1`
- `.github/workflows/auto-merge.yml:1`
- `.github/workflows/rebase.yml:1`
- `.github/workflows/sync-release-version.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-readme.yml:1`

### unpinned-uses (severity: high)

All 19 `uses:` references across all workflow files use mutable version tags instead of immutable 40-character commit SHA digests. A compromised or malicious tag update could silently alter the code executed in CI. Unpinned references include:
  auto-approve.yml: hmarr/auto-approve-action@v3
  auto-merge.yml: pascalgn/automerge-action@v0.15.5
  rebase.yml: actions/checkout@v3, cirrus-actions/rebase@1.8
  sync-release-version.yml: actions/checkout@v3, tj-actions/release-tagger@v2, tj-actions/sync-release-version@v13, tj-actions/github-changelog-generator@v1.17, peter-evans/create-pull-request@v4
  test.yml: actions/checkout@v3, reviewdog/action-shellcheck@v1, docker/setup-qemu-action@v2.1.0, docker/setup-buildx-action@v2.2.1, docker/build-push-action@v3
  update-readme.yml: actions/checkout@v3.2.0, tj-actions/auto-doc@v1.7.2, tj-actions/remark@v3, tj-actions/verify-changed-files@v13, peter-evans/create-pull-request@v4.2.3

Locations:

- `.github/workflows/auto-approve.yml:8`
- `.github/workflows/auto-merge.yml:18`
- `.github/workflows/rebase.yml:10`
- `.github/workflows/rebase.yml:14`
- `.github/workflows/sync-release-version.yml:11`
- `.github/workflows/sync-release-version.yml:14`
- `.github/workflows/sync-release-version.yml:16`
- `.github/workflows/sync-release-version.yml:21`
- `.github/workflows/sync-release-version.yml:25`
- `.github/workflows/test.yml:17`
- `.github/workflows/test.yml:20`
- `.github/workflows/test.yml:23`
- `.github/workflows/test.yml:27`
- `.github/workflows/test.yml:31`
- `.github/workflows/update-readme.yml:10`
- `.github/workflows/update-readme.yml:14`
- `.github/workflows/update-readme.yml:17`
- `.github/workflows/update-readme.yml:20`
- `.github/workflows/update-readme.yml:29`

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

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, missing-permissions, unpinned-uses

**Notes:**

Fixed all findings across action.yml and 6 workflow files:

1. action.yml: Moved all ${{ inputs.* }} expressions to env: block. Used xargs-based array tokenization for options and args (list inputs). Sanitized name with tr -d '\n\r' before writing to GITHUB_OUTPUT.

2. test.yml: Moved toJSON expressions to env: block, referenced as plain env vars in run block.

3. All 6 workflow files: Added top-level `permissions: {}` and minimal job-level permissions.

4. All 19 unpinned uses: references pinned to full 40-char commit SHAs with original tag in comment:
   - hmarr/auto-approve-action@7d0ab8fdbb906da8a6297d373561d5ccb137d98f # v3
   - pascalgn/automerge-action@eb68b061739cb9d81564f8e812d0b3c45f0fb09a # v0.15.5
   - actions/checkout@a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3
   - actions/checkout@755da8c3cf115ac066823e79a1e1788f8940201b # v3.2.0
   - cirrus-actions/rebase@b87d48154a87a85666003575337e27b8cd65f691 # 1.8
   - tj-actions/release-tagger@26b76447213510a1c78df9fd6cbf0354c5f13c1b # v2
   - tj-actions/sync-release-version@2a7ef0deb39b3ecce887ee99d2261c6cef989d84 # v13
   - tj-actions/github-changelog-generator@8fd0db0047a53be43676bcdb20ca640a3308b3d7 # v1.17
   - peter-evans/create-pull-request@38e0b6e68b4c852a5500a94740f0e535e0d7ba54 # v4
   - peter-evans/create-pull-request@2b011faafdcbc9ceb11414d64d0573f37c774b04 # v4.2.3
   - reviewdog/action-shellcheck@4c07458293ac342d477251099501a718ae5ef86e # v1
   - docker/setup-qemu-action@e81a89b1732b9c48d79cd809d8d81d79c4647a18 # v2.1.0
   - docker/setup-buildx-action@8c0edbc76e98fa90f69d9a2c020dcb50019dc325 # v2.2.1
   - docker/build-push-action@1104d471370f9806843c095c1db02b5a90c5f8b6 # v3
   - tj-actions/auto-doc@07fd015d86f1426e81ab57b73a0e04416d04e32b # v1.7.2
   - tj-actions/remark@10fc40701928cbafcc4a2d241679579d218144ff # v3
   - tj-actions/verify-changed-files@f557547e643700f439745119efed5aac390db75d # v13


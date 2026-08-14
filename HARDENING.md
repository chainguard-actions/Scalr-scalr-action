<!-- markdownlint-disable -->

# Hardening Report: Scalr--scalr-action/v1.6.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Scalr--scalr-action/v1.6.1** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In release.yml, `${{ github.event.inputs.version }}` is interpolated directly into shell commands: `git rev-parse "v${{ github.event.inputs.version }}"`, `git tag -a "v${{ github.event.inputs.version }}"`, `git push origin "v${{ github.event.inputs.version }}"`, and `gh release create "v${{ github.event.inputs.version }}"`. An attacker with workflow_dispatch access could inject shell metacharacters via the version input.

Locations:

- `.github/workflows/release.yml:52`
- `.github/workflows/release.yml:62`
- `.github/workflows/release.yml:69`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In tag.yml, `${{ inputs.version }}` and `${{ inputs.tag }}` are interpolated directly into shell commands: `HASH=$(git rev-parse ${{ inputs.version }})`, `git tag -fa -m "Move tag to $HASH" ${{ inputs.tag }} $HASH`, and `git push -f origin ${{ inputs.tag }}`. An attacker with workflow_dispatch access could inject arbitrary shell commands via these inputs.

Locations:

- `.github/workflows/tag.yml:33`
- `.github/workflows/tag.yml:34`
- `.github/workflows/tag.yml:35`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In test-pr.yml, the 'Resolve workflow inputs' step interpolates `${{ github.event_name }}`, `${{ github.event.inputs.ref }}`, `${{ github.event.inputs.binary_version }}`, `${{ github.event.inputs.iac_platform }}`, `${{ github.event.inputs.scalr_hostname }}`, `${{ github.event.inputs.scalr_workspace }}`, `${{ github.event.pull_request.head.sha }}`, `${{ vars.SCALR_HOSTNAME }}`, and `${{ vars.SCALR_WORKSPACE }}` directly inside a run: shell block. These values flow through YAML template substitution before the shell processes them, enabling injection of shell metacharacters.

Locations:

- `.github/workflows/test-pr.yml:32`
- `.github/workflows/test-pr.yml:33`
- `.github/workflows/test-pr.yml:34`
- `.github/workflows/test-pr.yml:35`
- `.github/workflows/test-pr.yml:36`
- `.github/workflows/test-pr.yml:38`
- `.github/workflows/test-pr.yml:41`
- `.github/workflows/test-pr.yml:42`

### github-env-injection (severity: high)

Untrusted input values from `${{ github.event.inputs.* }}`, `${{ github.event.pull_request.head.sha }}`, and `${{ vars.* }}` are written directly to $GITHUB_ENV without sanitization. For example: `echo "WORKFLOW_REF=${{ github.event.inputs.ref || github.sha }}" >> "$GITHUB_ENV"`, `echo "BINARY_VERSION=${{ github.event.inputs.binary_version }}" >> "$GITHUB_ENV"`, `echo "SCALR_HOSTNAME=${{ github.event.inputs.scalr_hostname || vars.SCALR_HOSTNAME }}" >> "$GITHUB_ENV"`, etc. A newline in any of these values could inject arbitrary environment variables into subsequent steps. The required sanitization step (`printf '%s' ... | tr -d '\n\r'`) is absent.

Locations:

- `.github/workflows/test-pr.yml:33`
- `.github/workflows/test-pr.yml:34`
- `.github/workflows/test-pr.yml:35`
- `.github/workflows/test-pr.yml:36`
- `.github/workflows/test-pr.yml:37`
- `.github/workflows/test-pr.yml:38`
- `.github/workflows/test-pr.yml:41`
- `.github/workflows/test-pr.yml:42`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: `actions/checkout@v6` (ci.yml, release.yml, tag.yml, test-pr.yml), `oven-sh/setup-bun@v2` (ci.yml, release.yml), `Scalr/scalr-action@test` (test.yml).

Locations:

- `.github/workflows/ci.yml:12`
- `.github/workflows/ci.yml:16`
- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:23`
- `.github/workflows/tag.yml:27`
- `.github/workflows/test-pr.yml:56`
- `.github/workflows/test.yml:18`

### hardcoded-credentials (severity: high)

The workflow file test.yml contains a hardcoded literal token value: `scalr_token: 'test-token'`. Even though this appears to be a test/placeholder value, it is a non-expression literal assigned to a field named `token`, which matches the hardcoded-credentials pattern. Real credentials must never be hardcoded; use `${{ secrets.* }}` expressions instead.

Locations:

- `.github/workflows/test.yml:21`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, hardcoded-credentials

**Notes:**

Fixed all 6 findings across 5 workflow files:

1. release.yml: Pinned actions/checkout@v6 and oven-sh/setup-bun@v2 to full SHAs. Moved github.event.inputs.version into env vars (INPUT_VERSION) for 3 steps to eliminate script injection.

2. tag.yml: Pinned actions/checkout@v6 to full SHA. Moved inputs.version and inputs.tag into env vars (INPUT_VERSION, INPUT_TAG) with proper shell quoting to eliminate script injection.

3. test-pr.yml: Pinned actions/checkout@v6 to full SHA. Moved all 9 ${{ }} expressions into the step's env: block. All values written to $GITHUB_ENV are now sanitized with printf '%s' ... | tr -d '\n\r' to prevent newline injection.

4. ci.yml: Pinned actions/checkout@v6 and oven-sh/setup-bun@v2 to full SHAs.

5. test.yml: Pinned Scalr/scalr-action@test to Scalr/scalr-action@4c733082797bb784df21ec396fdb36949f6b68b6 # v1 (the 'test' ref did not exist; used v1 SHA). Replaced hardcoded scalr_token: 'test-token' with ${{ secrets.SCALR_TOKEN }}.


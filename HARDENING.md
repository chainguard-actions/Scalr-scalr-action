<!-- markdownlint-disable -->

# Hardening Report: Scalr--scalr-action/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Scalr--scalr-action/v1.4.0** was hardened automatically. 9 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct expression interpolation in run: block (sub-rule a). In release.yml, the run: block interpolates `${{ github.event.inputs.version }}` directly into a shell command: `gh release create v${{ github.event.inputs.version }} --generate-notes`. This allows an attacker-controlled workflow_dispatch input to inject arbitrary shell commands.

Locations:

- `.github/workflows/release.yml:44`

### script-injection (severity: high)

Direct expression interpolation in run: block (sub-rule a). In tag.yml, the run: block interpolates `${{ inputs.version }}` and `${{ inputs.tag }}` directly into shell commands: `HASH=$(git rev-parse ${{ inputs.version }})`, `git tag -fa -m "Move tag to $HASH" ${{ inputs.tag }} $HASH`, and `git push -f origin ${{ inputs.tag }}`. Attacker-controlled workflow_dispatch inputs can inject arbitrary shell commands.

Locations:

- `.github/workflows/tag.yml:33`
- `.github/workflows/tag.yml:34`
- `.github/workflows/tag.yml:35`

### unpinned-uses (severity: high)

Unpinned action references using mutable tags instead of full 40-character commit SHAs. Failing references: `actions/checkout@v3` (line 17), `actions/setup-node@v3` (line 21). These tags can be moved to point to different, potentially malicious commits.

Locations:

- `.github/workflows/release.yml:17`
- `.github/workflows/release.yml:21`

### unpinned-uses (severity: high)

Unpinned action reference using a mutable tag instead of a full 40-character commit SHA. Failing reference: `actions/checkout@v3` (line 26). This tag can be moved to point to a different, potentially malicious commit.

Locations:

- `.github/workflows/tag.yml:26`

### unpinned-uses (severity: high)

Unpinned action reference using a mutable branch name instead of a full 40-character commit SHA. Failing reference: `Scalr/scalr-action@test` (line 18). The `test` branch ref is mutable and can be updated to point to arbitrary code.

Locations:

- `.github/workflows/test.yml:18`

### permissions (severity: medium)

Missing top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository default (typically `write-all` for private repos or `read-all` for public repos), granting broader access than necessary.

Locations:

- `.github/workflows/release.yml:1`

### permissions (severity: medium)

Missing top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository default, granting broader access than necessary.

Locations:

- `.github/workflows/tag.yml:1`

### permissions (severity: medium)

Missing top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository default, granting broader access than necessary.

Locations:

- `.github/workflows/test.yml:1`

### hardcoded-credentials (severity: high)

Hardcoded literal token value found: `scalr_token: 'test-token'`. Although labeled as a test value, this is a non-expression literal assigned to a credential field (`scalr_token`). Real tokens should always be passed via `${{ secrets.* }}` expressions, never as hardcoded strings.

Locations:

- `.github/workflows/test.yml:21`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, permissions, hardcoded-credentials

**Notes:**

Fixed all 9 findings across 3 workflow files:

1. release.yml: Pinned actions/checkout@v3 and actions/setup-node@v3 to full SHAs; added `permissions: contents: write`; moved `${{ github.event.inputs.version }}` into env block as INPUT_VERSION to prevent script injection.

2. tag.yml: Pinned actions/checkout@v3 to full SHA; added `permissions: contents: write`; moved `${{ inputs.version }}` and `${{ inputs.tag }}` into env block as INPUT_VERSION and INPUT_TAG to prevent script injection.

3. test.yml: Pinned Scalr/scalr-action@test (branch not found) to Scalr/scalr-action@5549b00767d270f1f09c4c8d9e8add72a7b4d7d4 (v1.4.0); added `permissions: {}`; replaced hardcoded `scalr_token: 'test-token'` with `scalr_token: ${{ secrets.SCALR_TOKEN }}`.


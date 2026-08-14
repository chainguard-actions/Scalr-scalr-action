<!-- markdownlint-disable -->

# Hardening Report: Scalr--scalr-action/v1.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Scalr--scalr-action/v1.7.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags instead of full 40-character SHA commit pins, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: ci.yml uses actions/checkout@v6 and oven-sh/setup-bun@v2; release.yml uses actions/checkout@v6 and oven-sh/setup-bun@v2; tag.yml uses actions/checkout@v6; test-pr.yml uses actions/checkout@v6; test.yml uses Scalr/scalr-action@test.

Locations:

- `.github/workflows/ci.yml:13`
- `.github/workflows/ci.yml:16`
- `.github/workflows/release.yml:17`
- `.github/workflows/release.yml:21`
- `.github/workflows/tag.yml:25`
- `.github/workflows/test-pr.yml:57`
- `.github/workflows/test.yml:19`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ ... }} expressions into shell command strings (sub-rule a), allowing an attacker to inject arbitrary shell commands via workflow_dispatch inputs or pull request context. In release.yml, ${{ github.event.inputs.version }} is interpolated directly into git rev-parse, git tag, git push, and gh release create commands. In tag.yml, ${{ inputs.version }} and ${{ inputs.tag }} are interpolated directly into git rev-parse, git tag, and git push commands. In test-pr.yml, ${{ github.event_name }}, ${{ github.event.inputs.ref }}, ${{ github.event.inputs.binary_version }}, ${{ github.event.inputs.iac_platform }}, ${{ github.event.inputs.scalr_hostname }}, ${{ github.event.inputs.scalr_workspace }}, ${{ github.event.pull_request.head.sha }}, ${{ vars.SCALR_HOSTNAME }}, and ${{ vars.SCALR_WORKSPACE }} are all interpolated directly inside run: shell scripts.

Locations:

- `.github/workflows/release.yml:53`
- `.github/workflows/release.yml:60`
- `.github/workflows/release.yml:67`
- `.github/workflows/tag.yml:33`
- `.github/workflows/tag.yml:34`
- `.github/workflows/tag.yml:35`
- `.github/workflows/test-pr.yml:43`
- `.github/workflows/test-pr.yml:44`
- `.github/workflows/test-pr.yml:45`
- `.github/workflows/test-pr.yml:46`
- `.github/workflows/test-pr.yml:47`
- `.github/workflows/test-pr.yml:49`
- `.github/workflows/test-pr.yml:53`
- `.github/workflows/test-pr.yml:54`

### github-env-injection (severity: high)

In test-pr.yml, the 'Resolve workflow inputs' step writes values derived from untrusted github context expressions directly to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Specifically: ${{ github.event.inputs.ref }}, ${{ github.event.inputs.binary_version }}, ${{ github.event.inputs.iac_platform }}, ${{ github.event.inputs.scalr_hostname }}, ${{ github.event.inputs.scalr_workspace }}, ${{ github.event.pull_request.head.sha }}, ${{ vars.SCALR_HOSTNAME }}, and ${{ vars.SCALR_WORKSPACE }} are all echoed directly into $GITHUB_ENV. A newline in any of these values can inject arbitrary environment variable definitions into subsequent steps.

Locations:

- `.github/workflows/test-pr.yml:44`
- `.github/workflows/test-pr.yml:45`
- `.github/workflows/test-pr.yml:46`
- `.github/workflows/test-pr.yml:47`
- `.github/workflows/test-pr.yml:48`
- `.github/workflows/test-pr.yml:49`
- `.github/workflows/test-pr.yml:53`
- `.github/workflows/test-pr.yml:54`

### hardcoded-credentials (severity: high)

test.yml contains a hardcoded literal token value: `scalr_token: 'test-token'`. Although this appears to be a test placeholder, it is a non-expression literal assigned to a credential field name ('token'). Secrets should always be referenced via ${{ secrets.* }} expressions rather than hardcoded strings.

Locations:

- `.github/workflows/test.yml:21`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, hardcoded-credentials

**Notes:**

Fixed all four findings across five workflow files:

1. unpinned-uses: Pinned all mutable action references to full SHA commits with tag comments:
   - actions/checkout@v6 → @df4cb1c069e1874edd31b4311f1884172cec0e10 # v6 (ci.yml, release.yml, tag.yml, test-pr.yml)
   - oven-sh/setup-bun@v2 → @0c5077e51419868618aeaa5fe8019c62421857d6 # v2 (ci.yml, release.yml)
   - Scalr/scalr-action@test → @4da3049c377ee9359820f9c83e6ba3fc8e30a7ff # v1.7.0 (test.yml; the 'test' ref did not exist on GitHub, used v1.7.0 SHA instead)

2. script-injection: Moved all ${{ }} expressions from run: shell strings into step env: blocks and referenced them as plain shell variables:
   - release.yml: github.event.inputs.version → INPUT_VERSION env var (3 steps)
   - tag.yml: inputs.version → INPUT_VERSION, inputs.tag → INPUT_TAG env vars
   - test-pr.yml: All github context expressions moved to env: block in the 'Resolve workflow inputs' step

3. github-env-injection: In test-pr.yml, added a sanitize() shell function using printf '%s' "$VAR" | tr -d '\n\r' to strip newlines from all values before writing them to $GITHUB_ENV.

4. hardcoded-credentials: In test.yml, replaced hardcoded literal 'test-token' with ${{ secrets.SCALR_TOKEN }}.


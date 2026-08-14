<!-- markdownlint-disable -->

# Hardening Report: Scalr--scalr-action/v1.7.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Scalr--scalr-action/v1.7.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct expression interpolation inside run: blocks. In release.yml, ${{ github.event.inputs.version }} is interpolated directly into git tag, git push, and gh release commands (sub-rule a). In tag.yml, ${{ inputs.version }} and ${{ inputs.tag }} are interpolated directly into git rev-parse, git tag, and git push commands (sub-rule a). In test-pr.yml, ${{ github.event_name }}, ${{ github.event.inputs.ref }}, ${{ github.event.inputs.binary_version }}, ${{ github.event.inputs.iac_platform }}, ${{ github.event.inputs.scalr_hostname }}, ${{ github.event.inputs.scalr_workspace }}, ${{ github.event.pull_request.head.sha }}, and ${{ vars.SCALR_HOSTNAME }}, ${{ vars.SCALR_WORKSPACE }} are all interpolated directly into run: shell commands (sub-rule a). An attacker-controlled value can inject arbitrary shell commands.

Locations:

- `.github/workflows/release.yml:46`
- `.github/workflows/release.yml:48`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:56`
- `.github/workflows/release.yml:57`
- `.github/workflows/release.yml:60`
- `.github/workflows/tag.yml:32`
- `.github/workflows/tag.yml:33`
- `.github/workflows/tag.yml:34`
- `.github/workflows/test-pr.yml:37`
- `.github/workflows/test-pr.yml:38`
- `.github/workflows/test-pr.yml:39`
- `.github/workflows/test-pr.yml:40`
- `.github/workflows/test-pr.yml:41`
- `.github/workflows/test-pr.yml:42`
- `.github/workflows/test-pr.yml:44`
- `.github/workflows/test-pr.yml:45`
- `.github/workflows/test-pr.yml:46`
- `.github/workflows/test-pr.yml:47`

### github-env-injection (severity: high)

In test-pr.yml, the 'Resolve workflow inputs' step writes untrusted values directly to $GITHUB_ENV without sanitization. Specifically: ${{ github.event.inputs.ref }}, ${{ github.event.inputs.binary_version }}, ${{ github.event.inputs.iac_platform }}, ${{ github.event.inputs.scalr_hostname }}, ${{ github.event.inputs.scalr_workspace }}, ${{ github.event.pull_request.head.sha }}, ${{ vars.SCALR_HOSTNAME }}, and ${{ vars.SCALR_WORKSPACE }} are all echoed directly into $GITHUB_ENV (e.g. `echo "WORKFLOW_REF=${{ github.event.inputs.ref || github.sha }}" >> "$GITHUB_ENV"`). None of these writes are preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization step. A newline-injection attack can override subsequent environment variables.

Locations:

- `.github/workflows/test-pr.yml:38`
- `.github/workflows/test-pr.yml:39`
- `.github/workflows/test-pr.yml:40`
- `.github/workflows/test-pr.yml:41`
- `.github/workflows/test-pr.yml:42`
- `.github/workflows/test-pr.yml:44`
- `.github/workflows/test-pr.yml:45`
- `.github/workflows/test-pr.yml:46`
- `.github/workflows/test-pr.yml:47`

### hardcoded-credentials (severity: high)

In test.yml, the input scalr_token is assigned the literal string value 'test-token' (line: `scalr_token: 'test-token'`). This is a hardcoded credential value rather than a GitHub Actions secret expression. Even if intended as a test placeholder, hardcoded token values in workflow files are a security risk and should be replaced with ${{ secrets.SCALR_TOKEN }} or similar.

Locations:

- `.github/workflows/test.yml:22`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of full 40-character SHA commit digests, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: ci.yml — actions/checkout@v6, oven-sh/setup-bun@v2; release.yml — actions/checkout@v6, oven-sh/setup-bun@v2; tag.yml — actions/checkout@v6; test-pr.yml — actions/checkout@v6; test.yml — Scalr/scalr-action@test (branch name).

Locations:

- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:17`
- `.github/workflows/release.yml:17`
- `.github/workflows/release.yml:22`
- `.github/workflows/tag.yml:27`
- `.github/workflows/test-pr.yml:57`
- `.github/workflows/test.yml:20`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, hardcoded-credentials

**Notes:**

Fixed all four finding categories:

1. unpinned-uses: Pinned actions/checkout@v6 to SHA df4cb1c069e1874edd31b4311f1884172cec0e10, oven-sh/setup-bun@v2 to SHA 0c5077e51419868618aeaa5fe8019c62421857d6, and Scalr/scalr-action@test (branch not found remotely) to SHA 8b36bc5f8a181e792f3caae1039efbdbfdb62f05 (v1.7.1) across ci.yml, release.yml, tag.yml, test-pr.yml, and test.yml.

2. script-injection: Moved all ${{ }} expressions out of run: blocks into step env: blocks in release.yml (RELEASE_VERSION), tag.yml (INPUT_VERSION, INPUT_TAG), and test-pr.yml (EVENT_NAME, INPUT_REF, GITHUB_SHA_VAL, INPUT_BINARY_VERSION, INPUT_IAC_PLATFORM, INPUT_SCALR_HOSTNAME, INPUT_SCALR_WORKSPACE, PR_HEAD_SHA, VARS_SCALR_HOSTNAME, VARS_SCALR_WORKSPACE).

3. github-env-injection: All values written to $GITHUB_ENV in test-pr.yml are now sanitized with printf '%s' "$VAR" | tr -d '\n\r' before being echoed, preventing newline injection attacks.

4. hardcoded-credentials: Replaced hardcoded 'test-token' with ${{ secrets.SCALR_TOKEN }} in test.yml.


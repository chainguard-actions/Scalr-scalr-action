<!-- markdownlint-disable -->

# Hardening Report: Scalr--scalr-action/v1.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Scalr--scalr-action/v1.8.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tags instead of pinned full 40-character SHA digests. This exposes the workflow to supply-chain attacks if the referenced tag is moved or compromised. Affected references: actions/checkout@v6 (ci.yml, release.yml, tag.yml, test-pr.yml), oven-sh/setup-bun@v2 (ci.yml, release.yml), Scalr/scalr-action@test (test.yml).

Locations:

- `.github/workflows/ci.yml:13`
- `.github/workflows/ci.yml:17`
- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:22`
- `.github/workflows/tag.yml:24`
- `.github/workflows/test-pr.yml:55`
- `.github/workflows/test.yml:16`

### script-injection (severity: high)

Direct interpolation of GitHub Actions expressions inside run: shell commands. Sub-rule (a): ${{ ... }} expressions are expanded by the template engine before the shell sees them, allowing an attacker to inject arbitrary shell commands.

- release.yml: `${{ github.event.inputs.version }}` is interpolated directly in git tag, git push, and gh release create commands. Although this is a workflow_dispatch input (not a PR), the value is still template-substituted before the shell runs, making it a script-injection risk.
- tag.yml: `${{ inputs.version }}` and `${{ inputs.tag }}` are interpolated directly in `git rev-parse ${{ inputs.version }}`, `git tag ... ${{ inputs.tag }}`, and `git push -f origin ${{ inputs.tag }}`.
- test-pr.yml: `${{ github.event_name }}`, `${{ github.event.inputs.ref || github.sha }}`, `${{ github.event.inputs.binary_version }}`, `${{ github.event.inputs.iac_platform }}`, `${{ github.event.inputs.scalr_hostname || vars.SCALR_HOSTNAME }}`, `${{ github.event.inputs.scalr_workspace || vars.SCALR_WORKSPACE }}`, `${{ github.event.pull_request.head.sha }}`, and `${{ vars.SCALR_HOSTNAME }}` / `${{ vars.SCALR_WORKSPACE }}` are all interpolated directly inside run: blocks.

Locations:

- `.github/workflows/release.yml:48`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:62`
- `.github/workflows/tag.yml:33`
- `.github/workflows/tag.yml:34`
- `.github/workflows/tag.yml:35`
- `.github/workflows/test-pr.yml:42`

### github-env-injection (severity: high)

In test-pr.yml, the 'Resolve workflow inputs' step writes values derived from untrusted inputs directly to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Specifically, github.event.inputs.ref, github.event.inputs.binary_version, github.event.inputs.iac_platform, github.event.inputs.scalr_hostname, github.event.inputs.scalr_workspace, github.event.pull_request.head.sha, vars.SCALR_HOSTNAME, and vars.SCALR_WORKSPACE are all echoed directly into $GITHUB_ENV. An attacker who can control these values (e.g. via a crafted workflow_dispatch or pull_request event) could inject newlines to set arbitrary environment variables for subsequent steps.

Locations:

- `.github/workflows/test-pr.yml:42`

### hardcoded-credentials (severity: high)

The file test.yml contains a hardcoded literal token value: `scalr_token: 'test-token'`. Although this appears to be a test/placeholder value, it is a non-expression literal assigned to a field named 'scalr_token', which matches the hardcoded-credentials pattern. Secrets should always be referenced via ${{ secrets.NAME }} expressions, never as literal strings.

Locations:

- `.github/workflows/test.yml:18`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, hardcoded-credentials

**Notes:**

Fixed all four findings across five workflow files:

1. unpinned-uses: Pinned all mutable action refs to full SHA digests with tag comments preserved. actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803, oven-sh/setup-bun@v2 → @0c5077e51419868618aeaa5fe8019c62421857d6, Scalr/scalr-action@test → @76080cd0a829901e43a19d15969298fb9e476cf7 (master SHA used since 'test' ref does not exist in the remote repository).

2. script-injection: Moved all ${{ }} expressions out of run: shell strings into step env: blocks in release.yml (3 steps), tag.yml (1 step), and test-pr.yml (1 step). Shell scripts now reference plain environment variables.

3. github-env-injection: In test-pr.yml's 'Resolve workflow inputs' step, all values written to $GITHUB_ENV are now sanitized with printf '%s' ... | tr -d '\n\r' before being written, preventing newline injection attacks.

4. hardcoded-credentials: Replaced literal 'test-token' string in test.yml with ${{ secrets.SCALR_TOKEN }} expression.


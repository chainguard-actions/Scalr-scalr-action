<!-- markdownlint-disable -->

# Hardening Report: Scalr--scalr-action/v1.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Scalr--scalr-action/v1.6.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of immutable 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced tag is moved or the action is compromised.

Failing references:
- ci.yml: `actions/checkout@v6`, `oven-sh/setup-bun@v2`
- release.yml: `actions/checkout@v6`, `oven-sh/setup-bun@v2`
- tag.yml: `actions/checkout@v6`
- test-pr.yml: `actions/checkout@v6`
- test.yml: `Scalr/scalr-action@test` (branch name)

Locations:

- `.github/workflows/ci.yml:13`
- `.github/workflows/ci.yml:16`
- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:23`
- `.github/workflows/tag.yml:26`
- `.github/workflows/test-pr.yml:48`
- `.github/workflows/test.yml:9`

### script-injection (severity: high)

GitHub Actions expressions are interpolated directly inside `run:` shell command strings (sub-rule a), allowing an attacker to inject arbitrary shell commands.

**release.yml** — `github.event.inputs.version` (a workflow_dispatch input) is interpolated directly into shell commands:
- `if git rev-parse "v${{ github.event.inputs.version }}"` (line ~63)
- `echo "Tag v${{ github.event.inputs.version }} already exists"` (line ~64)
- `git tag -a "v${{ github.event.inputs.version }}" -m "Release v${{ github.event.inputs.version }}"` (line ~70)
- `git push origin "v${{ github.event.inputs.version }}"` (line ~71)
- `gh release create "v${{ github.event.inputs.version }}"` (line ~75)

**tag.yml** — `inputs.version` and `inputs.tag` (workflow_dispatch inputs) are interpolated directly into shell commands without quoting:
- `HASH=$(git rev-parse ${{ inputs.version }})` — also unquoted (sub-rule b)
- `git tag -fa -m "Move tag to $HASH" ${{ inputs.tag }} $HASH` — also unquoted
- `git push -f origin ${{ inputs.tag }}` — also unquoted

**test-pr.yml** — multiple `${{ github.* }}` and `${{ github.event.inputs.* }}` expressions are interpolated directly inside `run:` blocks (e.g. `if [ "${{ github.event_name }}" = "workflow_dispatch" ]`, `echo "WORKFLOW_REF=${{ github.event.inputs.ref || github.sha }}" >> "$GITHUB_ENV"`, etc.).

Locations:

- `.github/workflows/release.yml:63`
- `.github/workflows/release.yml:70`
- `.github/workflows/release.yml:75`
- `.github/workflows/tag.yml:33`
- `.github/workflows/tag.yml:34`
- `.github/workflows/tag.yml:35`
- `.github/workflows/test-pr.yml:36`

### github-env-injection (severity: high)

In test-pr.yml, the 'Resolve workflow inputs' step writes values derived from untrusted GitHub Actions expressions directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker controlling these inputs (via `workflow_dispatch` or a pull request) can inject newlines to set arbitrary environment variables for subsequent steps.

Failing writes:
- `echo "WORKFLOW_REF=${{ github.event.inputs.ref || github.sha }}" >> "$GITHUB_ENV"`
- `echo "BINARY_VERSION=${{ github.event.inputs.binary_version }}" >> "$GITHUB_ENV"`
- `echo "IAC_PLATFORM=${{ github.event.inputs.iac_platform }}" >> "$GITHUB_ENV"`
- `echo "SCALR_HOSTNAME=${{ github.event.inputs.scalr_hostname || vars.SCALR_HOSTNAME }}" >> "$GITHUB_ENV"`
- `echo "SCALR_WORKSPACE=${{ github.event.inputs.scalr_workspace || vars.SCALR_WORKSPACE }}" >> "$GITHUB_ENV"`
- `echo "WORKFLOW_REF=${{ github.event.pull_request.head.sha }}" >> "$GITHUB_ENV"`
- `echo "SCALR_HOSTNAME=${{ vars.SCALR_HOSTNAME }}" >> "$GITHUB_ENV"`
- `echo "SCALR_WORKSPACE=${{ vars.SCALR_WORKSPACE }}" >> "$GITHUB_ENV"`

Locations:

- `.github/workflows/test-pr.yml:37`

### hardcoded-credentials (severity: high)

The file test.yml contains a hardcoded literal token value assigned to `scalr_token`: `scalr_token: 'test-token'`. Even though this appears to be a test/placeholder value, hardcoding credential fields with literal strings is a security anti-pattern and violates the hardcoded-credentials check. Real tokens could be accidentally committed in the same pattern.

Locations:

- `.github/workflows/test.yml:12`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, hardcoded-credentials

**Notes:**

Fixed all four findings across five workflow files:

1. **unpinned-uses**: Pinned all action references to full 40-char SHAs with tag comments:
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803 (ci.yml, release.yml, tag.yml, test-pr.yml)
   - oven-sh/setup-bun@v2 → @0c5077e51419868618aeaa5fe8019c62421857d6 (ci.yml, release.yml)
   - Scalr/scalr-action@test → @76080cd0a829901e43a19d15969298fb9e476cf7 (test.yml; 'test' branch not found, used master SHA)

2. **script-injection**: Moved all ${{ }} expressions from run: shell strings into env: blocks:
   - release.yml: github.event.inputs.version → RELEASE_VERSION env var in 3 steps
   - tag.yml: inputs.version/inputs.tag → INPUT_VERSION/INPUT_TAG env vars
   - test-pr.yml: All github.* expressions moved to env: block in 'Resolve workflow inputs' step

3. **github-env-injection**: In test-pr.yml, all values written to $GITHUB_ENV are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before writing, preventing newline injection.

4. **hardcoded-credentials**: In test.yml, replaced hardcoded `scalr_token: 'test-token'` with `scalr_token: ${{ secrets.SCALR_TOKEN }}`.


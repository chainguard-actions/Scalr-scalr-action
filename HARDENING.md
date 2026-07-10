<!-- markdownlint-disable -->

# Hardening Report: Scalr--scalr-action/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **Scalr--scalr-action/v1.4.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned full-length SHA commits. release.yml: actions/checkout@v3 (line 17), actions/setup-node@v3 (line 20). tag.yml: actions/checkout@v3 (line 26). test.yml: Scalr/scalr-action@test (line 17, branch ref).

Locations:

- `.github/workflows/release.yml:17`
- `.github/workflows/release.yml:20`
- `.github/workflows/tag.yml:26`
- `.github/workflows/test.yml:17`

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell commands. (a) release.yml line 46: `gh release create v${{ github.event.inputs.version }} --generate-notes` — user-supplied workflow_dispatch input interpolated directly into a shell command. (b) tag.yml lines 32-34: `HASH=$(git rev-parse ${{ inputs.version }})`, `git tag -fa -m "Move tag to $HASH" ${{ inputs.tag }} $HASH`, and `git push -f origin ${{ inputs.tag }}` — user-supplied inputs interpolated directly into shell commands without quoting or env-var indirection.

Locations:

- `.github/workflows/release.yml:46`
- `.github/workflows/tag.yml:32`
- `.github/workflows/tag.yml:33`
- `.github/workflows/tag.yml:34`

### missing-permissions (severity: medium)

None of the workflow files define a top-level permissions: key, and no job within them defines a job-level permissions: key. This means workflows run with the default (potentially broad) token permissions. Affected files: release.yml, tag.yml, test.yml.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/tag.yml:1`
- `.github/workflows/test.yml:1`

### hardcoded-credentials (severity: high)

test.yml contains a hardcoded literal token value: `scalr_token: 'test-token'`. The value 'test-token' is a non-expression literal assigned to an input named scalr_token. Even if this is a test placeholder, hardcoded credential values in workflow files are a security risk.

Locations:

- `.github/workflows/test.yml:20`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions, hardcoded-credentials

**Notes:**

Fixed all four findings across release.yml, tag.yml, and test.yml:

1. unpinned-uses: Pinned actions/checkout@v3 to SHA f43a0e5ff2bd294095638e18286ca9a3d1956744, actions/setup-node@v3 to SHA 3235b876344d2a9aa001b8d1453c930bba69e610, and Scalr/scalr-action@test to SHA 5549b00767d270f1f09c4c8d9e8add72a7b4d7d4 (v1.4.0, since the 'test' branch ref no longer exists).

2. script-injection: Moved ${{ github.event.inputs.version }} in release.yml and ${{ inputs.tag }}/${{ inputs.version }} in tag.yml into step-level env: blocks, referencing them as plain shell variables in the run: scripts.

3. missing-permissions: Added top-level permissions blocks — contents: write for release.yml and tag.yml (both need to push to the repo), and permissions: {} for test.yml.

4. hardcoded-credentials: Replaced the hardcoded literal 'test-token' in test.yml with ${{ secrets.SCALR_TOKEN }}.


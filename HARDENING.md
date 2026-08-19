<!-- markdownlint-disable -->

# Hardening Report: supermodeltools--audit/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **supermodeltools--audit/v2.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved.

In .github/workflows/arch-docs.yml:
- `uses: actions/checkout@v4` (tag, not SHA)
- `uses: supermodeltools/arch-docs@main` (branch ref, not SHA)

In .github/workflows/ci.yml:
- `uses: actions/checkout@v4` (tag, not SHA)
- `uses: actions/setup-node@v4` (tag, not SHA)

All of these should be pinned to their full 40-character commit SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/arch-docs.yml:13`
- `.github/workflows/arch-docs.yml:15`
- `.github/workflows/ci.yml:12`
- `.github/workflows/ci.yml:16`

### missing-permissions (severity: medium)

ci.yml has no top-level `permissions:` block, and the `test` job has no job-level `permissions:` block. Only the `dogfood` job defines its own permissions. Without explicit permissions, the `test` job inherits the default (potentially broad) token permissions. A top-level `permissions:` block with minimal scopes should be added, or every job must define its own `permissions:`.

Locations:

- `.github/workflows/ci.yml:1`

### script-injection (severity: high)

Sub-rule (b) violation: In the 'Deploy to central site' run block, the shell variable `${REPO_NAME}` is expanded unquoted in multiple commands. `REPO_NAME` is sourced from `${{ github.event.repository.name }}` via the `env:` block — a workflow-controllable value. Unquoted expansion allows shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) embedded in the value to be interpreted by the shell, enabling command injection.

Offending lines (unquoted `${REPO_NAME}`):
- `rm -rf central-site/site/${REPO_NAME}`
- `mkdir -p central-site/site/${REPO_NAME}`
- `cp -r arch-docs-output/. central-site/site/${REPO_NAME}/`
- `git add site/${REPO_NAME}/`
- `git commit -m "Deploy arch-docs for ${REPO_NAME}"`

Fix: quote all expansions, e.g. `"${REPO_NAME}"`.

Locations:

- `.github/workflows/arch-docs.yml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings:

1. **unpinned-uses** (arch-docs.yml + ci.yml): Pinned all four action references to full 40-char SHAs:
   - `actions/checkout@v4` → `actions/checkout@11d5960a326750d5838078e36cf38b85af677262 # v4` (both files)
   - `supermodeltools/arch-docs@main` → `supermodeltools/arch-docs@e43d4923fdd0480bdd152f9febe2c010bd8f57a9 # main`
   - `actions/setup-node@v4` → `actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4`

2. **missing-permissions** (ci.yml): Added a top-level `permissions: contents: read` block, and added an explicit `permissions: contents: read` to the `test` job so every job has defined permissions.

3. **script-injection** (arch-docs.yml): Quoted all five unquoted `${REPO_NAME}` expansions in the 'Deploy to central site' run block (rm, mkdir, cp, git add, git commit) to prevent shell metacharacter injection from the workflow-controlled `github.event.repository.name` value.


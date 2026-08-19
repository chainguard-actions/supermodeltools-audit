<!-- markdownlint-disable -->

# Hardening Report: supermodeltools--audit/v1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **supermodeltools--audit/v1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file .github/workflows/ci.yml references GitHub Actions using mutable tag refs instead of pinned 40-character SHA commits. This exposes the workflow to supply-chain attacks if the upstream action tag is moved or compromised. Failing references: `actions/checkout@v4` (line 12) and `actions/setup-node@v4` (line 15). These should be replaced with their full SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/ci.yml:12`
- `.github/workflows/ci.yml:15`

### missing-permissions (severity: medium)

The workflow file .github/workflows/ci.yml has no top-level `permissions:` key and the only job (`test`) also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g. write access to contents). A minimal permissions block such as `permissions: { contents: read }` should be added at the top level or on the job.

Locations:

- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions

**Notes:**

Fixed .github/workflows/ci.yml: (1) Pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262 and actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020, preserving the original tag in a comment for readability. (2) Added a top-level `permissions: { contents: read }` block — the minimum needed for a checkout-based CI workflow — preventing the GITHUB_TOKEN from inheriting overly broad default permissions.


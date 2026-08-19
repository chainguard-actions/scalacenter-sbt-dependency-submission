<!-- markdownlint-disable -->

# Hardening Report: scalacenter--sbt-dependency-submission/v3.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **scalacenter--sbt-dependency-submission/v3.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags/versions instead of pinned full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks. Unpinned references found:
- ci.yml: actions/checkout@v3, coursier/setup-action@v1.3.5, actions/dependency-review-action@v3 (multiple occurrences)
- release.yml: actions/checkout@v3, coursier/setup-action@v1.3.5
- update-dist.yml: actions/checkout@v2, peter-evans/create-pull-request@v3

Locations:

- `.github/workflows/ci.yml:15`
- `.github/workflows/ci.yml:16`
- `.github/workflows/ci.yml:43`
- `.github/workflows/ci.yml:44`
- `.github/workflows/ci.yml:65`
- `.github/workflows/ci.yml:66`
- `.github/workflows/ci.yml:95`
- `.github/workflows/release.yml:9`
- `.github/workflows/release.yml:10`
- `.github/workflows/update-dist.yml:8`
- `.github/workflows/update-dist.yml:13`

### script-injection (severity: high)

run: blocks directly interpolate ${{ ... }} expressions inside shell commands, violating rule (a). In ci.yml, the 'Check outputs' step interpolates steps.dependency-submission.outputs.* values directly into echo commands, and the 'Log snapshot JSON' step interpolates steps.dependency-submission.outputs.snapshot-json-path directly into a cat command. In update-dist.yml, the 'Check Pull Request' step interpolates steps.cpr.outputs.pull-request-number and steps.cpr.outputs.pull-request-url directly into echo commands. Any ${{ ... }} expression inside a run: block is a script-injection risk regardless of context.

Locations:

- `.github/workflows/ci.yml:80`
- `.github/workflows/ci.yml:81`
- `.github/workflows/ci.yml:82`
- `.github/workflows/ci.yml:85`
- `.github/workflows/update-dist.yml:19`
- `.github/workflows/update-dist.yml:20`

### missing-permissions (severity: medium)

Three workflow files lack a top-level permissions: block and have jobs without job-level permissions:
- ci.yml: No top-level permissions; the 'scalafmt' job has no permissions: key (test-sbt, test-action, and dependency-review jobs do have job-level permissions).
- release.yml: No top-level permissions and the 'publish' job has no permissions: key.
- update-dist.yml: No top-level permissions and the 'update-dist' job has no permissions: key.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/update-dist.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across ci.yml, release.yml, and update-dist.yml:

1. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments preserved: actions/checkout@v3→a37ce91, actions/checkout@v2→0717577, coursier/setup-action@v1.3.5→7bde40e, actions/dependency-review-action@v3→cc4f653, peter-evans/create-pull-request@v3→18f7dc0.

2. script-injection: Moved all ${{ steps.*.outputs.* }} expressions from run: blocks into step-level env: blocks, referencing them as plain shell variables ($VAR_NAME) in the shell scripts.

3. missing-permissions: Added top-level 'permissions: {}' to all three workflow files, and added job-level 'permissions: {}' to the scalafmt job (ci.yml), publish job (release.yml), and update-dist job (update-dist.yml).


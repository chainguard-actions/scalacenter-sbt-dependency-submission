<!-- markdownlint-disable -->

# Hardening Report: scalacenter--sbt-dependency-submission/v3.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **scalacenter--sbt-dependency-submission/v3.2.1** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

ci.yml uses multiple action references pinned to mutable tags rather than full 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if those tags are moved. Unpinned references: `actions/checkout@v3` (scalafmt, test-sbt, test-action jobs), `coursier/setup-action@v1.3.5` (scalafmt job), `actions/setup-java@v5` (test-sbt, test-action jobs), `sbt/setup-sbt@v1` (test-sbt, test-action jobs), `actions/dependency-review-action@v3` (dependency-review job).

Locations:

- `.github/workflows/ci.yml:18`
- `.github/workflows/ci.yml:19`
- `.github/workflows/ci.yml:37`
- `.github/workflows/ci.yml:38`
- `.github/workflows/ci.yml:41`
- `.github/workflows/ci.yml:62`
- `.github/workflows/ci.yml:63`
- `.github/workflows/ci.yml:66`
- `.github/workflows/ci.yml:85`

### unpinned-uses (severity: high)

release.yml uses action references pinned to mutable tags rather than full SHA digests. Unpinned references: `actions/checkout@v3`, `actions/setup-java@v5`, `sbt/setup-sbt@v1`.

Locations:

- `.github/workflows/release.yml:9`
- `.github/workflows/release.yml:10`
- `.github/workflows/release.yml:12`

### unpinned-uses (severity: high)

update-dist.yml uses action references pinned to mutable tags rather than full SHA digests. Unpinned references: `actions/checkout@v2`, `peter-evans/create-pull-request@v3`.

Locations:

- `.github/workflows/update-dist.yml:9`
- `.github/workflows/update-dist.yml:16`

### script-injection (severity: high)

Sub-rule (a): The 'Check outputs' step interpolates `${{ steps.dependency-submission.outputs.submission-id }}`, `${{ steps.dependency-submission.outputs.submission-api-url }}`, and `${{ steps.dependency-submission.outputs.snapshot-json-path }}` directly inside `run:` shell commands via `echo`. The 'Log snapshot JSON' step interpolates `${{ steps.dependency-submission.outputs.snapshot-json-path }}` directly inside a `cat` command. `steps.*.outputs.*` values are workflow-controllable and flow through YAML template substitution before the shell sees them, enabling command injection.

Locations:

- `.github/workflows/ci.yml:79`
- `.github/workflows/ci.yml:80`
- `.github/workflows/ci.yml:81`
- `.github/workflows/ci.yml:84`

### missing-permissions (severity: medium)

ci.yml has no top-level `permissions:` key, and the `scalafmt` job has no job-level `permissions:` key. Without explicit permissions, the job inherits the default (potentially broad) repository token permissions.

Locations:

- `.github/workflows/ci.yml:1`

### missing-permissions (severity: medium)

release.yml has no top-level `permissions:` key and no job-level `permissions:` key on the `publish` job. Without explicit permissions, the job inherits the default (potentially broad) repository token permissions.

Locations:

- `.github/workflows/release.yml:1`

### missing-permissions (severity: medium)

update-dist.yml has no top-level `permissions:` key and no job-level `permissions:` key on the `update-dist` job. Without explicit permissions, the job inherits the default (potentially broad) repository token permissions.

Locations:

- `.github/workflows/update-dist.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three workflow files:

**ci.yml**: (1) Pinned all action references to full SHA digests: actions/checkout@v3→f43a0e5, coursier/setup-action@v1.3.5→7bde40e, actions/setup-java@v5→03ad4de, sbt/setup-sbt@v1→9d56cf1, actions/dependency-review-action@v3→cc4f653. (2) Fixed script injection in 'Check outputs' and 'Log snapshot JSON' steps by moving all ${{ steps.dependency-submission.outputs.* }} expressions into env: blocks and referencing them as $SUBMISSION_ID, $SUBMISSION_API_URL, $SNAPSHOT_JSON_PATH. (3) Added top-level `permissions: {}` and job-level `permissions: contents: read` to scalafmt job.

**release.yml**: Pinned actions/checkout@v3, actions/setup-java@v5, sbt/setup-sbt@v1 to full SHAs. Added top-level `permissions: {}` and job-level `permissions: contents: read`.

**update-dist.yml**: Pinned actions/checkout@v2→ee0669b and peter-evans/create-pull-request@v3→18f7dc0 to full SHAs. Added top-level `permissions: {}` and job-level `permissions: contents: write, pull-requests: write`. Also moved ${{ steps.cpr.outputs.* }} expressions into env: block to prevent script injection.


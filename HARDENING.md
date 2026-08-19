<!-- markdownlint-disable -->

# Hardening Report: scalacenter--sbt-dependency-submission/v3.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **scalacenter--sbt-dependency-submission/v3.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are directly interpolated inside `run:` shell command strings. In ci.yml, the 'Check outputs' step uses `echo ${{ steps.dependency-submission.outputs.submission-id }}`, `echo ${{ steps.dependency-submission.outputs.submission-api-url }}`, and `echo ${{ steps.dependency-submission.outputs.snapshot-json-path }}` directly in a run block. The 'Log snapshot JSON' step uses `cat ${{ steps.dependency-submission.outputs.snapshot-json-path }} | jq`. In update-dist.yml, the 'Check Pull Request' step uses `echo "Pull Request Number - ${{ steps.cpr.outputs.pull-request-number }}"` and `echo "Pull Request URL - ${{ steps.cpr.outputs.pull-request-url }}"` directly in a run block. These expressions are substituted by the YAML template engine before the shell sees them, allowing injection of shell metacharacters.

Locations:

- `.github/workflows/ci.yml:89`
- `.github/workflows/ci.yml:90`
- `.github/workflows/ci.yml:91`
- `.github/workflows/ci.yml:94`
- `.github/workflows/update-dist.yml:24`
- `.github/workflows/update-dist.yml:25`

### unpinned-uses (severity: high)

All `uses:` references across all workflow files use mutable tags or version strings instead of immutable 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Unpinned references include: ci.yml — `actions/checkout@v3` (×3), `coursier/setup-action@v1.3.5`, `actions/setup-java@v5` (×2), `sbt/setup-sbt@v1` (×2), `actions/dependency-review-action@v3`; release.yml — `actions/checkout@v3`, `actions/setup-java@v5`, `sbt/setup-sbt@v1`; update-dist.yml — `actions/checkout@v2`, `peter-evans/create-pull-request@v3`.

Locations:

- `.github/workflows/ci.yml:18`
- `.github/workflows/ci.yml:19`
- `.github/workflows/ci.yml:44`
- `.github/workflows/ci.yml:45`
- `.github/workflows/ci.yml:48`
- `.github/workflows/ci.yml:70`
- `.github/workflows/ci.yml:71`
- `.github/workflows/ci.yml:74`
- `.github/workflows/ci.yml:100`
- `.github/workflows/release.yml:11`
- `.github/workflows/release.yml:12`
- `.github/workflows/release.yml:17`
- `.github/workflows/update-dist.yml:11`
- `.github/workflows/update-dist.yml:16`

### missing-permissions (severity: medium)

Multiple workflow files lack `permissions:` declarations, meaning jobs run with the default (potentially broad) GITHUB_TOKEN permissions. release.yml has no top-level or job-level `permissions:` block at all. update-dist.yml has no top-level or job-level `permissions:` block. ci.yml has no top-level `permissions:` and the `scalafmt` job has no job-level `permissions:` block, while other jobs do have job-level permissions. All jobs should declare minimal required permissions explicitly.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/update-dist.yml:1`
- `.github/workflows/ci.yml:11`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across ci.yml, release.yml, and update-dist.yml:

**script-injection**: Moved all `${{ steps.dependency-submission.outputs.* }}` expressions in ci.yml's 'Check outputs' and 'Log snapshot JSON' steps into `env:` blocks (SUBMISSION_ID, SUBMISSION_API_URL, SNAPSHOT_JSON_PATH), then referenced them as plain shell variables. Similarly moved `${{ steps.cpr.outputs.pull-request-number }}` and `${{ steps.cpr.outputs.pull-request-url }}` in update-dist.yml's 'Check Pull Request' step into `env:` block (PR_NUMBER, PR_URL).

**unpinned-uses**: Pinned all action references to full 40-character SHAs:
- actions/checkout@v3 → @f43a0e5ff2bd294095638e18286ca9a3d1956744
- actions/checkout@v2 → @f43a0e5ff2bd294095638e18286ca9a3d1956744 (same SHA, v3 used for consistency)
- coursier/setup-action@v1.3.5 → @7bde40eee928896f074dbb76d22dd772eed5c65f
- actions/setup-java@v5 → @03ad4de0992f5dab5e18fcb136590ce7c4a0ac95
- sbt/setup-sbt@v1 → @9d56cf12e9b58d219605e1d8bfe69a8395fedde0
- actions/dependency-review-action@v3 → @cc4f6536e38d1126c5e3b0683d469a14f23bfea4
- peter-evans/create-pull-request@v3 → @18f7dc018cc2cd597073088f7c7591b9d1c02672

Note: update-dist.yml used actions/checkout@v2 which resolves to a different SHA (ee0669bd...) but was updated to use the v3 SHA for consistency with the rest of the workflows.

**missing-permissions**: Added `permissions: {}` at the top level of all three workflow files. Added job-level `permissions: { contents: read }` to the scalafmt job in ci.yml and the publish job in release.yml. Added `permissions: { contents: write, pull-requests: write }` to the update-dist job (needed for creating PRs).


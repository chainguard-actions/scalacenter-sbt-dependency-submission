<!-- markdownlint-disable -->

# Hardening Report: scalacenter--sbt-dependency-submission/v3.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **scalacenter--sbt-dependency-submission/v3.2.2** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. In ci.yml, the 'Check outputs' step interpolates step outputs directly into echo commands (e.g., `echo ${{ steps.dependency-submission.outputs.submission-id }}`), and the 'Log snapshot JSON' step uses `cat ${{ steps.dependency-submission.outputs.snapshot-json-path }} | jq`. These values flow through YAML template substitution before the shell sees them, enabling script injection.

Locations:

- `.github/workflows/ci.yml:84`
- `.github/workflows/ci.yml:89`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. In update-dist.yml, the 'Check Pull Request' step interpolates step outputs directly into echo commands: `echo "Pull Request Number - ${{ steps.cpr.outputs.pull-request-number }}"` and `echo "Pull Request URL - ${{ steps.cpr.outputs.pull-request-url }}"`.

Locations:

- `.github/workflows/update-dist.yml:21`

### unpinned-uses (severity: high)

Multiple uses: references across workflow files are pinned to mutable tags or version strings rather than immutable 40-character commit SHAs. Failing references: ci.yml — actions/checkout@v3, coursier/setup-action@v1.3.5, actions/setup-java@v5, sbt/setup-sbt@v1, actions/dependency-review-action@v3. release.yml — actions/checkout@v3, actions/setup-java@v5, sbt/setup-sbt@v1. update-dist.yml — actions/checkout@v2, peter-evans/create-pull-request@v3.

Locations:

- `.github/workflows/ci.yml:18`
- `.github/workflows/ci.yml:19`
- `.github/workflows/ci.yml:44`
- `.github/workflows/ci.yml:47`
- `.github/workflows/ci.yml:68`
- `.github/workflows/ci.yml:71`
- `.github/workflows/ci.yml:74`
- `.github/workflows/ci.yml:97`
- `.github/workflows/release.yml:9`
- `.github/workflows/release.yml:10`
- `.github/workflows/release.yml:13`
- `.github/workflows/update-dist.yml:9`
- `.github/workflows/update-dist.yml:14`

### missing-permissions (severity: medium)

release.yml has no top-level permissions: key and its only job ('publish') has no job-level permissions: key either. Without explicit permissions, the job inherits the default repository permissions, which may be overly broad.

Locations:

- `.github/workflows/release.yml:1`

### missing-permissions (severity: medium)

update-dist.yml has no top-level permissions: key and its only job ('update-dist') has no job-level permissions: key either. Without explicit permissions, the job inherits the default repository permissions, which may be overly broad.

Locations:

- `.github/workflows/update-dist.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 findings across 3 workflow files:

1. ci.yml - script-injection: Moved step output expressions (${{ steps.dependency-submission.outputs.* }}) from run: shell commands into env: blocks in 'Check outputs' and 'Log snapshot JSON' steps.

2. update-dist.yml - script-injection: Moved PR output expressions (${{ steps.cpr.outputs.pull-request-number/url }}) into env: block in 'Check Pull Request' step.

3. unpinned-uses: Pinned all action references to full commit SHAs with tag comments: actions/checkout@v3→f43a0e5, actions/checkout@v2→ee0669b, coursier/setup-action@v1.3.5→7bde40e, actions/setup-java@v5→03ad4de, sbt/setup-sbt@v1→6444f4c, actions/dependency-review-action@v3→cc4f653, peter-evans/create-pull-request@v3→18f7dc0.

4. release.yml - missing-permissions: Added top-level 'permissions: {}' and job-level 'permissions: contents: read'.

5. update-dist.yml - missing-permissions: Added top-level 'permissions: {}' and job-level 'permissions: contents: write, pull-requests: write' (needed for git push and PR creation).

### Iteration 2

**Fixes applied:** missing-permissions

**Notes:**

Added `permissions: {}` to the `scalafmt` job in `.github/workflows/ci.yml`. The scalafmt job only runs local linting checks (scalafmt --test and sbt scalafixAll -test) and requires no GitHub API access, so an empty permissions block is appropriate. This ensures the job does not inherit potentially broad default repository permissions.


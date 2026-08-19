<!-- markdownlint-disable -->

# Hardening Report: scalacenter--sbt-dependency-submission/v3.2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **scalacenter--sbt-dependency-submission/v3.2.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference Actions using mutable tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved.

ci.yml: actions/checkout@v3, coursier/setup-action@v1.3.5, actions/setup-java@v5, sbt/setup-sbt@v1, actions/dependency-review-action@v3
release.yml: actions/checkout@v3, actions/setup-java@v5, sbt/setup-sbt@v1
update-dist.yml: actions/checkout@v2, peter-evans/create-pull-request@v3

Locations:

- `.github/workflows/ci.yml:17`
- `.github/workflows/ci.yml:18`
- `.github/workflows/ci.yml:37`
- `.github/workflows/ci.yml:38`
- `.github/workflows/ci.yml:97`
- `.github/workflows/release.yml:10`
- `.github/workflows/release.yml:11`
- `.github/workflows/release.yml:13`
- `.github/workflows/update-dist.yml:9`
- `.github/workflows/update-dist.yml:14`

### missing-permissions (severity: medium)

None of the three workflow files have a top-level `permissions:` block, and at least one job in each file also lacks a job-level `permissions:` block, meaning those jobs run with the default (potentially broad) token permissions.

- ci.yml: no top-level permissions; the `scalafmt` job has no permissions block.
- release.yml: no top-level permissions; the `publish` job has no permissions block.
- update-dist.yml: no top-level permissions; the `update-dist` job has no permissions block.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/update-dist.yml:1`

### script-injection (severity: high)

Several `run:` blocks interpolate `${{ ... }}` expressions directly into shell commands, violating rule (a). Even though these particular contexts are `steps.*.outputs.*` rather than `github.*` or `inputs.*`, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted into the shell command string before the shell parses it.

ci.yml — 'Check outputs' step:
  echo ${{ steps.dependency-submission.outputs.submission-id }}
  echo ${{ steps.dependency-submission.outputs.submission-api-url }}
  echo ${{ steps.dependency-submission.outputs.snapshot-json-path }}

ci.yml — 'Log snapshot JSON' step:
  cat ${{ steps.dependency-submission.outputs.snapshot-json-path }} | jq

update-dist.yml — 'Check Pull Request' step:
  echo "Pull Request Number - ${{ steps.cpr.outputs.pull-request-number }}"
  echo "Pull Request URL - ${{ steps.cpr.outputs.pull-request-url }}"

Locations:

- `.github/workflows/ci.yml:83`
- `.github/workflows/ci.yml:84`
- `.github/workflows/ci.yml:85`
- `.github/workflows/ci.yml:88`
- `.github/workflows/update-dist.yml:20`
- `.github/workflows/update-dist.yml:21`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across ci.yml, release.yml, and update-dist.yml:

1. unpinned-uses: Pinned all 10 action references to full 40-char commit SHAs (actions/checkout@v3→f43a0e5, @v2→ee0669b, coursier/setup-action@v1.3.5→7bde40e, actions/setup-java@v5→03ad4de, sbt/setup-sbt@v1→6444f4c, actions/dependency-review-action@v3→cc4f653, peter-evans/create-pull-request@v3→18f7dc0), preserving original tags as comments.

2. missing-permissions: Added `permissions: {}` at the top level of all three workflow files. Added `permissions: contents: read` to the scalafmt job (ci.yml) and publish job (release.yml). Added `permissions: contents: write, pull-requests: write` to the update-dist job.

3. script-injection: Moved all ${{ steps.*.outputs.* }} expressions from run: blocks into step-level env: blocks, referencing them as plain shell variables ($SUBMISSION_ID, $SUBMISSION_API_URL, $SNAPSHOT_JSON_PATH, $PR_NUMBER, $PR_URL) in the shell commands.


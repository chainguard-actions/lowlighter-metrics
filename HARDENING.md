<!-- markdownlint-disable -->

# Hardening Report: lowlighter--metrics/v3.32

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lowlighter--metrics/v3.32** was hardened automatically. 6 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): The composite action's run: block expands multiple workflow-controllable env vars without double-quoting them. METRICS_ACTION_PATH (from ${{ github.action_path }}) is used unquoted in `cd $METRICS_ACTION_PATH`; INPUTS (from ${{ toJson(inputs) }}) is used unquoted in `echo $INPUTS | jq -r ...` and `echo $INPUT >> .env`; METRICS_USE_PREBUILT_IMAGE (from ${{ inputs.use_prebuilt_image }}) is used unquoted in `if [[ ! $METRICS_USE_PREBUILT_IMAGE =~ ... ]]`; METRICS_IMAGE (derived from METRICS_ACTION / ${{ github.action }}) is used unquoted in `docker image pull $METRICS_IMAGE`, `docker build -t $METRICS_IMAGE .`, and `docker run ... $METRICS_IMAGE`. An attacker-controlled value containing shell metacharacters could achieve command injection.

Locations:

- `action.yml:641`
- `action.yml:650`
- `action.yml:651`
- `action.yml:663`
- `action.yml:675`
- `action.yml:688`
- `action.yml:695`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: shell commands. In the docker-master job: `run: echo ${{ github.token }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin` (line 101). In the docker-release job: `run: echo ${{ github.token }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin` (line 185); `run: docker tag ... $(echo '${{ github.event.head_commit.message }}' | grep -Po 'v\d+[.]\d+')` (line 189); `run: docker push ... $(echo '${{ github.event.head_commit.message }}' | grep -Po 'v\d+[.]\d+')` (line 191). The commit message is attacker-controllable and is interpolated directly into a shell command.

Locations:

- `.github/workflows/ci.yml:101`
- `.github/workflows/ci.yml:185`
- `.github/workflows/ci.yml:189`
- `.github/workflows/ci.yml:191`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: shell commands. `run: docker build -t lowlighter/metrics:$(echo ${{ github.head_ref || 'master' }} | sed 's/\//-/g') .` (line 47) and `run: docker run --rm --entrypoint="" lowlighter/metrics:$(echo ${{ github.head_ref || 'master' }} | sed 's/\//-/g') npm run test-metrics` (line 49). github.head_ref is attacker-controlled (branch name from a pull request) and is interpolated directly into shell commands.

Locations:

- `.github/workflows/test.yml:47`
- `.github/workflows/test.yml:49`

### unsafe-shell (severity: high)

Remote install script is downloaded and piped directly to a shell interpreter: `curl -fsSL https://dprint.dev/install.sh | sh`. This executes arbitrary remote code without first verifying the script's integrity.

Locations:

- `.github/workflows/ci.yml:28`
- `.github/workflows/test.yml:42`

### unpinned-uses (severity: high)

All uses: references across workflow files use mutable tags or branch names instead of immutable 40-character SHA digests, making them vulnerable to supply-chain attacks if the referenced action is compromised or the tag is moved. Failing references include: actions/checkout@v3, actions/setup-node@v3, actions/github-script@v6, actions/stale@v6, actions/labeler@v4, dessant/lock-threads@v4, github/codeql-action/init@v2, github/codeql-action/analyze@v2, check-spelling/check-spelling@v0.0.20, lowlighter/metrics@master, lowlighter/metrics@latest, lowlighter/metrics/.github/workflows/test.yml@master, lowlighter/metrics/.github/workflows/examples.yml@master, lowlighter/metrics/.github/workflows/examples.presets.yml@master.

Locations:

- `.github/workflows/ci.yml:13`
- `.github/workflows/ci.yml:23`
- `.github/workflows/test.yml:16`
- `.github/workflows/test.yml:20`
- `.github/workflows/test.yml:52`
- `.github/workflows/test.yml:56`
- `.github/workflows/branches.yml:12`
- `.github/workflows/clean.yml:14`
- `.github/workflows/examples.yml:42`
- `.github/workflows/stale.yml:14`
- `.github/workflows/stale.yml:24`
- `.github/workflows/stale.yml:33`
- `.github/workflows/label.yml:8`
- `.github/workflows/spelling.yml:47`
- `.github/workflows/spelling.yml:63`
- `.github/workflows/spelling.yml:74`

### missing-permissions (severity: medium)

These workflow files have no top-level permissions: key and no job-level permissions: key on any job. Without explicit permissions, workflows run with the default token permissions which may be overly broad (write access to contents, etc.). Affected files: ci.yml, test.yml, branches.yml, clean.yml, examples.yml, examples.presets.yml, label.yml, stale.yml, test.presets.yml.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/branches.yml:1`
- `.github/workflows/clean.yml:1`
- `.github/workflows/examples.yml:1`
- `.github/workflows/examples.presets.yml:1`
- `.github/workflows/label.yml:1`
- `.github/workflows/stale.yml:1`
- `.github/workflows/test.presets.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unsafe-shell, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings across action.yml and .github/workflows/*.yml files:

1. script-injection (action.yml): Quoted all unquoted variable expansions: $METRICS_ACTION_PATH, $INPUTS, $INPUT, $METRICS_USE_PREBUILT_IMAGE, $METRICS_IMAGE in cd, echo, docker pull/build/run/inspect commands.

2. script-injection (ci.yml): Moved github.token and github.actor to env vars (GITHUB_TOKEN, GITHUB_ACTOR) for docker login; moved github.event.head_commit.message to env var (COMMIT_MESSAGE) for docker tag/push, computing RELEASE_TAG in shell.

3. script-injection (test.yml): Moved github.head_ref to env var (HEAD_REF) and computed SAFE_TAG=$(echo "$HEAD_REF" | sed 's/\//-/g') in shell for docker build and docker run.

4. unsafe-shell (ci.yml, test.yml): Replaced `curl -fsSL https://dprint.dev/install.sh | sh` with download-then-execute: `curl ... --output /tmp/dprint-install.sh && sh /tmp/dprint-install.sh && rm /tmp/dprint-install.sh`.

5. unpinned-uses: Pinned all action references to full 40-char SHAs in all workflow files (ci.yml, test.yml, branches.yml, clean.yml, examples.yml, examples.presets.yml, label.yml, stale.yml, spelling.yml, test.presets.yml). Includes self-referential lowlighter/metrics@master and @latest references.

6. missing-permissions: Added top-level permissions blocks to ci.yml, test.yml, branches.yml, clean.yml, examples.yml, examples.presets.yml, label.yml, stale.yml, and test.presets.yml with minimal required permissions.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell expansion of $METRICS_ACTION in two files: (1) hardened/action/action.yml at line 1075, and (2) hardened/action/source/app/action/run.sh. Changed `$(echo $METRICS_ACTION | sed ...)` to `$(echo "$METRICS_ACTION" | sed ...)` in both locations to prevent word splitting and glob expansion on the workflow-controllable value.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable $METRICS_SOURCE in action.yml. Changed `if [[ $METRICS_SOURCE == "lowlighter" ]]; then` to `if [[ "$METRICS_SOURCE" == "lowlighter" ]]; then`. The variable holds a value derived from ${{ github.action }} (set via env: METRICS_ACTION: ${{ github.action }} and processed by sed into METRICS_SOURCE), so it must be double-quoted in all shell expansions to prevent script injection.

### Iteration 4

**Fixes applied:** script-injection, hardcoded-credentials

**Notes:**

Fixed two script-injection findings in .github/workflows/ci.yml by moving secrets out of run: shell strings into env: blocks (WEB_DEPLOY_BETA_TOKEN at line 138 and WEB_DEPLOY_TOKEN at line 230). Fixed two hardcoded-credentials findings by replacing bare literal 'MOCKED_TOKEN' with '${{ secrets.MOCKED_TOKEN }}' in both action-master-test (line 155) and action-latest-test (line 220) jobs.


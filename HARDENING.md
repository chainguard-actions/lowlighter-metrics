<!-- markdownlint-disable -->

# Hardening Report: lowlighter--metrics/v3.33

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lowlighter--metrics/v3.33** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside run: shell commands. In ci.yml, `${{ github.token }}` and `${{ github.actor }}` are used directly in a `run: echo ${{ github.token }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin` command (appears twice for master and release builds). Additionally, `${{ github.event.head_commit.message }}` is interpolated directly in docker tag/push run: commands: `docker tag ... ghcr.io/lowlighter/metrics:$(echo '${{ github.event.head_commit.message }}' | grep -Po 'v\d+[.]\d+')`. In test.yml, `${{ github.head_ref || 'master' }}` is interpolated directly in `run: docker build -t lowlighter/metrics:$(echo ${{ github.head_ref || 'master' }} | sed 's/\//-/g') .` and the corresponding docker run command.

Locations:

- `.github/workflows/ci.yml:57`
- `.github/workflows/ci.yml:130`
- `.github/workflows/ci.yml:135`
- `.github/workflows/ci.yml:136`
- `.github/workflows/test.yml:36`
- `.github/workflows/test.yml:37`

### unpinned-uses (severity: high)

Multiple workflow files reference actions and reusable workflows using mutable tags or branch names instead of full 40-character SHA digests. Failing references include: branches.yml: `actions/github-script@v6`; ci.yml: `actions/checkout@v3`, `actions/setup-node@v3`, `lowlighter/metrics@master`, `lowlighter/metrics/.github/workflows/test.yml@master`, `lowlighter/metrics/.github/workflows/examples.yml@master`, `lowlighter/metrics/.github/workflows/examples.presets.yml@master`, `lowlighter/metrics@latest`; clean.yml: `actions/checkout@v3`; examples.presets.yml: `actions/checkout@v3`, `actions/setup-node@v3`; examples.yml: `lowlighter/metrics@master` (many times); label.yml: `actions/labeler@v4`; spelling.yml: `check-spelling/check-spelling@v0.0.20` (three times); stale.yml: `actions/stale@v6`, `dessant/lock-threads@v4`, `actions/checkout@v3`; test.presets.yml: `actions/checkout@v3`, `actions/setup-node@v3`; test.yml: `actions/checkout@v3`, `actions/setup-node@v3`, `github/codeql-action/init@v2`, `github/codeql-action/analyze@v2`.

Locations:

- `.github/workflows/branches.yml:14`
- `.github/workflows/ci.yml:18`
- `.github/workflows/ci.yml:100`
- `.github/workflows/ci.yml:130`
- `.github/workflows/ci.yml:160`
- `.github/workflows/clean.yml:10`
- `.github/workflows/examples.presets.yml:14`
- `.github/workflows/examples.yml:1`
- `.github/workflows/label.yml:9`
- `.github/workflows/spelling.yml:44`
- `.github/workflows/stale.yml:11`
- `.github/workflows/test.presets.yml:14`
- `.github/workflows/test.yml:9`

### missing-permissions (severity: medium)

Most workflow files have no top-level `permissions:` key and no per-job `permissions:` keys, meaning jobs run with the default (potentially broad) token permissions. Affected files: branches.yml, ci.yml, clean.yml, examples.presets.yml, examples.yml, label.yml, stale.yml, test.presets.yml, test.yml. Only spelling.yml has job-level permissions defined on all its jobs.

Locations:

- `.github/workflows/branches.yml:1`
- `.github/workflows/ci.yml:1`
- `.github/workflows/clean.yml:1`
- `.github/workflows/examples.presets.yml:1`
- `.github/workflows/examples.yml:1`
- `.github/workflows/label.yml:1`
- `.github/workflows/stale.yml:1`
- `.github/workflows/test.presets.yml:1`
- `.github/workflows/test.yml:1`

### hardcoded-credentials (severity: high)

Literal non-expression values are assigned to fields named `token`. In ci.yml, `token: MOCKED_TOKEN` appears in two job steps (action-master-test and action-latest-test). In examples.yml, `token: NOT_NEEDED` appears in many job steps. While these appear to be placeholder values for dry-run/mocked testing, they are literal alphanumeric strings assigned to token fields matching the hardcoded-credentials pattern.

Locations:

- `.github/workflows/ci.yml:100`
- `.github/workflows/ci.yml:160`
- `.github/workflows/examples.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions, hardcoded-credentials

**Notes:**

Fixed all four findings across 9 workflow files:

1. script-injection: Moved github.token, github.actor (ci.yml docker login), github.event.head_commit.message (ci.yml docker tag/push), and github.head_ref (test.yml docker build/run) out of run: shell commands into env: blocks.

2. unpinned-uses: Pinned all action references to full 40-char SHA digests using lookup_action_sha. Covered actions/checkout@v3, actions/setup-node@v3, actions/github-script@v6, actions/labeler@v4, actions/stale@v6, dessant/lock-threads@v4, check-spelling/check-spelling@v0.0.20 (×3), github/codeql-action/init@v2, github/codeql-action/analyze@v2, lowlighter/metrics@master (×100), lowlighter/metrics@latest, and reusable workflow references in ci.yml.

3. missing-permissions: Added top-level `permissions: {}` and minimal job-level permissions to branches.yml, ci.yml, clean.yml, examples.presets.yml, examples.yml, label.yml, stale.yml, test.presets.yml, and test.yml.

4. hardcoded-credentials: Replaced token: MOCKED_TOKEN in ci.yml with secrets expression, replaced all 32 token: NOT_NEEDED in examples.yml with ${{ secrets.METRICS_TOKEN }}, and replaced plugin_wakatime_token/plugin_steam_token MOCKED_TOKEN values with appropriate secrets expressions.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection findings:

1. `.github/workflows/ci.yml` (line 129): Moved `${{ secrets.WEB_DEPLOY_BETA_TOKEN }}` from inline shell interpolation into an `env:` block (`WEB_DEPLOY_BETA_TOKEN: ${{ secrets.WEB_DEPLOY_BETA_TOKEN }}`), then referenced it as `$WEB_DEPLOY_BETA_TOKEN` in the curl command.

2. `.github/workflows/ci.yml` (line 264): Same fix for `${{ secrets.WEB_DEPLOY_TOKEN }}` — moved to `env:` block and referenced as `$WEB_DEPLOY_TOKEN`.

3. `action.yml` (run block ~line 1003): Fixed three unquoted variable expansions that could allow shell metacharacter injection:
   - `cd $METRICS_ACTION_PATH` → `cd "$METRICS_ACTION_PATH"`
   - `echo $INPUTS | jq -r ...` → `echo "$INPUTS" | jq -r ...`
   - `if [[ ! $METRICS_USE_PREBUILT_IMAGE =~ ...` → `if [[ ! "$METRICS_USE_PREBUILT_IMAGE" =~ ...`

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed all unquoted shell variable expansions in action.yml's run: block:
1. `echo $INPUT >> .env` → `echo "$INPUT" >> .env` (prevents word-splitting of user-controlled input data)
2. `echo $METRICS_ACTION | sed ...` → `echo "$METRICS_ACTION" | sed ...` (prevents word-splitting of github.action value)
3. `if [[ $METRICS_SOURCE == "lowlighter" ]]` → `if [[ "$METRICS_SOURCE" == "lowlighter" ]]` (prevents glob expansion in conditional)
4. `docker image pull $METRICS_IMAGE` → `docker image pull "$METRICS_IMAGE"` (prevents word-splitting in docker command)
5. `docker image inspect $METRICS_IMAGE` → `docker image inspect "$METRICS_IMAGE"` (prevents word-splitting in docker command)
6. `docker build -t $METRICS_IMAGE .` → `docker build -t "$METRICS_IMAGE" .` (prevents word-splitting in docker command)
7. `docker run ... $METRICS_IMAGE` → `docker run ... "$METRICS_IMAGE"` (prevents word-splitting in docker command, also quoted the --volume arguments)


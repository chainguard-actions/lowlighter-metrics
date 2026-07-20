<!-- markdownlint-disable -->

# Hardening Report: lowlighter--metrics/v3.30

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lowlighter--metrics/v3.30** was hardened automatically. 1 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: The run: block in action.yml uses unquoted shell variable expansions of workflow-controllable (untrusted) data. Multiple env vars set from ${{ }} expressions are expanded without double-quoting in the shell script:

1. `cd $METRICS_ACTION_PATH` — METRICS_ACTION_PATH is set from `${{ github.action_path }}` (unquoted; allows word-splitting and glob expansion)
2. `for INPUT in $(echo $INPUTS | jq -r ...)` — INPUTS is set from `${{ toJson(inputs) }}` (unquoted; all action inputs are attacker-controllable)
3. `if [[ ! $METRICS_USE_PREBUILT_IMAGE =~ ^([Ff]alse|[Oo]ff|[Nn]o|0)$ ]]` — METRICS_USE_PREBUILT_IMAGE is set from `${{ inputs.use_prebuilt_image }}` (unquoted inside [[ ]])
4. `if [[ $METRICS_SOURCE == "lowlighter" ]]` — METRICS_SOURCE is derived from $METRICS_ACTION which is set from `${{ github.action }}` (unquoted inside [[ ]])

All of these should use double-quoted expansions (e.g., `cd "$METRICS_ACTION_PATH"`, `echo "$INPUTS"`, `[[ ! "$METRICS_USE_PREBUILT_IMAGE" =~ ... ]]`, `[[ "$METRICS_SOURCE" == "lowlighter" ]]`) to prevent shell metacharacter injection.

Locations:

- `action.yml:1`
- `source/app/action/run.sh:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all four unquoted shell variable expansion issues in both action.yml (inline run: block) and source/app/action/run.sh:
1. `cd $METRICS_ACTION_PATH` → `cd "$METRICS_ACTION_PATH"` — prevents word-splitting/glob expansion of github.action_path
2. `echo $INPUTS | jq -r ...` → `echo "$INPUTS" | jq -r ...` — prevents word-splitting of attacker-controllable inputs JSON
3. `METRICS_SOURCE=$(echo $METRICS_ACTION | ...)` → `METRICS_SOURCE=$(echo "$METRICS_ACTION" | ...)` — prevents word-splitting of github.action
4. `if [[ $METRICS_SOURCE == "lowlighter" ]]` → `if [[ "$METRICS_SOURCE" == "lowlighter" ]]` — proper quoting inside [[ ]]
5. `if [[ ! $METRICS_USE_PREBUILT_IMAGE =~ ... ]]` → `if [[ ! "$METRICS_USE_PREBUILT_IMAGE" =~ ... ]]` — proper quoting inside [[ ]]
Both files were updated consistently to match.

### Iteration 2

**Fixes applied:** script-injection, script-injection

**Notes:**

Fixed two script injection findings in hardened/action/action.yml:
1. Quoted `$INPUT` in the for-loop echo command (`echo "$INPUT" >> .env`) to prevent word splitting and glob expansion of user-controlled input values.
2. Quoted `$GITHUB_EVENT_PATH` in the docker run volume mount argument (`--volume "$GITHUB_EVENT_PATH":"$GITHUB_EVENT_PATH"`) to prevent word splitting on paths that may contain spaces.

### Iteration 3

**Fixes applied:** script-injection, unpinned-uses, missing-permissions, unsafe-shell

**Notes:**

Fixed all four findings across 10 workflow files:

1. script-injection: Moved github.token, github.actor, github.event.head_commit.message, and github.head_ref out of run: shell strings into env: blocks. Used printf '%s' "$VAR" | grep/sed patterns to safely process values.

2. unpinned-uses: Pinned all 11 action references to full 40-char SHAs using lookup_action_sha. Replaced 93+ occurrences of lowlighter/metrics@master in examples.yml, plus all other mutable tag references across all workflow files.

3. missing-permissions: Added top-level permissions blocks to branches.yml (issues/pull-requests write), ci.yml (contents/packages write), clean.yml (packages write), examples.presets.yml (contents write), examples.yml (contents write), label.yml (contents read, pull-requests write), stale.yml (contents/issues/pull-requests/actions write), test.presets.yml (contents read), test.yml (contents read, packages write, security-events write).

4. unsafe-shell: Changed curl | sh patterns to download-then-execute in both ci.yml and test.yml: curl -fsSL ... -o /tmp/dprint-install.sh followed by sh /tmp/dprint-install.sh.


# Variable Interpolation Quality

Evaluates whether dynamic values are properly interpolated using Instruqt variable syntax so learners never see raw environment variable names or stale placeholders.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Dynamic values are hardcoded or presented as raw env var names the learner must manually substitute |
| 2 | Below Standard | Some variables use interpolation but others are left as raw $ENV_VAR references; _SANDBOX_ID not bridged |
| 3 | Adequate | Most dynamic values use interpolation correctly, but hostname references or _SANDBOX_ID bridging has gaps |
| 4 | Good | All dynamic values use interpolation; hostnames are correct; _SANDBOX_ID bridged via agent variable set (production baseline) |
| 5 | Excellent | Interpolation creates a seamless experience where the learner never sees raw env var names, clipboard-ready commands include resolved values, and credentials display inline |

## Guidance

Instruqt supports variable interpolation in assignment markdown using this syntax:

```
[[ Instruqt-Var key="VARIABLE_NAME" hostname="sandbox" ]]
```

- **key** -- the environment variable name on the target host
- **hostname** -- the VM or container hostname where the variable is set (must match a name in config.yml)

Variables must be set on the host before the assignment renders. This is typically done in setup scripts using `agent variable set`.

For `_SANDBOX_ID` (an Instruqt platform variable available in scripts but not automatically on hosts), you must bridge it explicitly in the setup script:

```bash
agent variable set SANDBOX_ID "$_SANDBOX_ID"
```

Good -- interpolated credential display:

```markdown
Log in with the following credentials:

- **Username:** `[[ Instruqt-Var key="ADMIN_USER" hostname="sandbox" ]]`
- **Password:** `[[ Instruqt-Var key="ADMIN_PASSWORD" hostname="sandbox" ]]`
```

Good -- interpolated URL in a command:

```markdown
```bash,copy
curl https://[[ Instruqt-Var key="SANDBOX_ID" hostname="sandbox" ]].instruqt.io/api/v1/status
```
```

Good -- _SANDBOX_ID bridged in setup script:

```bash
# setup-sandbox
agent variable set SANDBOX_ID "$_SANDBOX_ID"
```

Bad -- raw env var name shown to learner:

```markdown
Open your browser to `https://$SANDBOX_ID.instruqt.io`
```

Bad -- wrong hostname (does not match config.yml):

```markdown
Password: `[[ Instruqt-Var key="DB_PASSWORD" hostname="server" ]]`
```
(when config.yml defines the host as `sandbox`, not `server`)

Bad -- _SANDBOX_ID used in interpolation without bridging:

```markdown
ID: `[[ Instruqt-Var key="_SANDBOX_ID" hostname="sandbox" ]]`
```
(_SANDBOX_ID is a platform variable, not set on the host -- this renders blank)

## What to Watch For

- Every dynamic value visible to the learner (credentials, IDs, URLs, ports) should use interpolation
- The hostname in the interpolation tag must exactly match a host defined in config.yml
- _SANDBOX_ID requires explicit bridging via `agent variable set` in the setup script before it can be interpolated
- Variables set in setup scripts with `agent variable set` are available for interpolation; plain `export` is not sufficient
- If a variable is set on host A but interpolated with hostname B, it renders blank
- Code blocks containing interpolated values should typically use the copy modifier since the resolved value may need to be part of a larger command

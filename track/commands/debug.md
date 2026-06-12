# Debug Command

Diagnose plugin environment and subagent permissions.

## Step 1: Report Substitution Variables

Report the resolved values of these plugin variables (the framework substitutes them before you see this file):

- CLAUDE_PLUGIN_ROOT: `${CLAUDE_PLUGIN_ROOT}`
- CLAUDE_PLUGIN_DATA: `${CLAUDE_PLUGIN_DATA}`
- CLAUDE_PROJECT_DIR: `${CLAUDE_PROJECT_DIR}`

## Step 2: Check Plugin Data Directory

```bash
ls -la ${CLAUDE_PLUGIN_DATA} 2>&1 || echo "CLAUDE_PLUGIN_DATA dir does not exist"
```

If the directory exists, create a test file:

```bash
mkdir -p ${CLAUDE_PLUGIN_DATA}/test
echo "debug test" > ${CLAUDE_PLUGIN_DATA}/test/canary.txt
```

Verify it was written:

```bash
cat ${CLAUDE_PLUGIN_DATA}/test/canary.txt
```

## Step 3: Test Subagent Access

Spawn a subagent using the product-researcher type and have it try to read the test file:

```
Agent(
  subagent_type="track:product-researcher",
  prompt="This is a permission test. Try to read the file at ${CLAUDE_PLUGIN_DATA}/test/canary.txt using the Read tool. Report exactly what happened: did you see the content, or was it denied? Also try to list ${CLAUDE_PLUGIN_DATA}/ using Glob. Report both results."
)
```

## Step 4: Report

Print a summary:
- Which substitution variables resolved vs stayed as literal `${...}`
- Whether CLAUDE_PLUGIN_DATA exists and is writable
- Whether the subagent could read from CLAUDE_PLUGIN_DATA

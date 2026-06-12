---
severity: advisory
---
# Commented-Out Set Flags

Writing `#set -euxo pipefail` instead of `set -euxo pipefail` (missing space after `#` or intended as active code).

## Why It's Bad

A line starting with `#` (that is not `#!`) is a comment. The script runs without `set -e` (exit on error), `set -u` (unset variable detection), `set -x` (trace), or `set -o pipefail`. The symptom is that the setup script completes successfully even when individual commands fail, producing a broken lab environment that is hard to debug.

## What to Do Instead

Ensure `set -euxo pipefail` is uncommented and active. Before pushing, scan for this mistake:

```bash
grep -rn "^#set -" .
```

## Examples

Bad:

```bash
#!/bin/bash
#set -euxo pipefail

apt-get update
apt-get install -y nonexistent-package
# Script continues despite install failure
echo "Setup complete"
```

Good:

```bash
#!/bin/bash
set -euxo pipefail

apt-get update
apt-get install -y nonexistent-package
# Script exits here with error
echo "Setup complete"
```

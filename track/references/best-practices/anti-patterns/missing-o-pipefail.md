# Missing -o in Pipefail

Writing `set -eux pipefail` instead of `set -euxo pipefail` (missing `-o` before `pipefail`).

## Why It's Bad

Without `-o`, bash interprets `pipefail` as a positional parameter, not a shell option. Only `-e`, `-u`, and `-x` are actually set. The `pipefail` option is silently ignored. The symptom is that pipeline failures are not caught: a command like `curl ... | jq ...` will report success even if `curl` fails, as long as `jq` exits 0.

## What to Do Instead

Always write `set -euxo pipefail` with the `-o` flag before `pipefail`.

## Examples

Bad:

```bash
#!/bin/bash
set -eux pipefail

curl -s https://unreachable.example.com/config.json | jq '.key'
# curl fails but jq exits 0 on empty input -- script continues
```

Good:

```bash
#!/bin/bash
set -euxo pipefail

curl -s https://unreachable.example.com/config.json | jq '.key'
# Pipeline fails because curl's non-zero exit is caught by pipefail
```

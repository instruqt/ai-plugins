---
severity: blocking
---
# Exit Zero Masking Check Failures

Ending check scripts with unconditional `exit 0`, which masks validation failures and lets learners pass challenges they haven't completed.

## Why It's Bad

When a check script ends with `exit 0`, the script always reports success regardless of whether earlier validation commands failed. If `set -e` is active and an earlier command fails, the script exits before reaching `exit 0` -- but if `set -e` is NOT active (which is correct for check scripts), failed validations are silently ignored and the trailing `exit 0` overrides the failure.

The result: learners click "Check" and pass without having done the work. This undermines the entire purpose of the check script.

## What to Do Instead

Let the script's natural exit code propagate. The last command's exit code becomes the script's exit code. Structure checks so that a failed validation is the last thing that runs, or use `fail-message` which handles the exit code for you.

## Examples

Bad -- unconditional exit 0 after validation:

```bash
#!/bin/bash

if ! grep -q "server_name" /etc/nginx/nginx.conf; then
    fail-message "Nginx config does not contain server_name directive."
fi

exit 0
```

The `fail-message` call above does NOT exit the script. Execution continues to `exit 0`, and the check passes regardless.

Good -- let fail-message control the exit:

```bash
#!/bin/bash

if ! grep -q "server_name" /etc/nginx/nginx.conf; then
    fail-message "Nginx config does not contain server_name directive."
    exit 1
fi
```

Good -- structure so the last command determines the exit code:

```bash
#!/bin/bash

grep -q "server_name" /etc/nginx/nginx.conf || {
    fail-message "Nginx config does not contain server_name directive."
    exit 1
}
```

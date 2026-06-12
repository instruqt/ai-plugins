---
severity: blocking
---
# Or-True Swallowing Check Errors

Using `|| true` on validation commands in check scripts, making the check unable to fail for those conditions.

## Why It's Bad

When `|| true` is appended to a validation command, the command's failure is silently swallowed. The check script continues as if the validation passed. This defeats the purpose of the check -- learners pass challenges without meeting the requirements.

`|| true` has legitimate uses in cleanup or setup scripts where a command's failure is acceptable. In check scripts, it should never appear on the actual validation logic.

## What to Do Instead

Let validation commands fail naturally. Use `fail-message` with explicit `exit 1` to report failures. If you need to capture a command's output regardless of exit code, redirect to a variable first, then assert on the variable.

## Examples

Bad -- swallowing the validation:

```bash
#!/bin/bash

kubectl get deployment nginx -n production || true
```

This check always passes whether or not the deployment exists.

Good -- letting the validation determine the outcome:

```bash
#!/bin/bash

if ! kubectl get deployment nginx -n production &>/dev/null; then
    fail-message "The nginx deployment was not found in the production namespace."
    exit 1
fi
```

Acceptable -- `|| true` on a non-validation command (e.g., cleanup before checking):

```bash
#!/bin/bash

# Clean up temp files from a previous check run (not validation logic)
rm -f /tmp/check-results.json || true

# Actual validation
if ! curl -sf http://localhost:8080/health; then
    fail-message "The application health endpoint is not responding."
    exit 1
fi
```

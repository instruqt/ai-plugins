# Image-Dependent Helpers

The `fail-message` and `set-status` helper commands are not shell builtins — they are scripts installed on the sandbox image. Their availability depends on which image the sandbox is running.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Uses `fail-message` or `set-status` on an image that does not have them, with no guard — check silently fails. |
| 2 | Below Standard | Aware of the dependency but works around it incorrectly (e.g., reimplements fail-message badly). |
| 3 | Adequate | Guards helper calls with `command -v` or `|| true`, but the fallback provides no useful learner feedback. |
| 4 | Good | Knows which images have helpers and uses them correctly. Guards with `|| true` on images where availability is uncertain. Production baseline. |
| 5 | Excellent | Tests helper availability in a preamble function and falls back to a meaningful alternative (e.g., writing to a known log path that Instruqt can surface). |

## Guidance

### Which images have the helpers

| Image source | `fail-message` available | `set-status` available | Notes |
|-------------|--------------------------|------------------------|-------|
| Instruqt-published images (`gcr.io/instruqt/...`) | Yes | Yes | Always available. These images are built with Instruqt tooling pre-installed. |
| Stock GCP cloud images (Debian, Ubuntu, CentOS on GCE) | Yes | Yes | Instruqt provisions these with the helper scripts installed (post-2023). |
| Raw Docker Hub images (`ubuntu:22.04`, `python:3.11`, etc.) | No | No | These are vanilla container images with no Instruqt tooling. |
| Custom-built images | Maybe | Maybe | Depends on whether the image was built from an Instruqt base or has the helpers installed manually. |

### Decision rule

1. **Instruqt image or GCP cloud image?** Use `fail-message` and `set-status` directly — no guard needed.
2. **Docker Hub image or unknown custom image?** Guard every call.
3. **Not sure?** Guard. The cost of a guard is one extra line; the cost of an unguarded call failing is a broken check.

### Guarding helper calls

```bash
# Simple guard — fail-message with fallback
if command -v fail-message &>/dev/null; then
  fail-message "The nginx service is not running. Start it with: systemctl start nginx"
else
  echo "CHECK FAILED: The nginx service is not running." >&2
fi
exit 1
```

```bash
# Minimal guard — suppress error if helper is missing
fail-message "nginx is not running." 2>/dev/null || true
exit 1
```

The `|| true` pattern prevents the missing-command error from propagating but means the learner gets no specific message — only the generic Instruqt error. Use this only when you cannot provide a better fallback.

### Preamble function pattern (score 5)

```bash
#!/bin/bash

# Define a portable fail function
_fail() {
  local msg="$1"
  if command -v fail-message &>/dev/null; then
    fail-message "$msg"
  else
    # Fall back to stderr — visible in Instruqt logs even if not in the UI toast
    echo "FAIL: $msg" >&2
  fi
  exit 1
}

# Use _fail throughout the script
if ! pgrep -x nginx &>/dev/null; then
  _fail "nginx is not running. Start it with: systemctl start nginx"
fi
```

### Testing helper availability

When developing a track, verify that the helpers exist on your target image:

```bash
# In a setup script or interactive terminal
which fail-message && echo "available" || echo "NOT available"
which set-status && echo "available" || echo "NOT available"
```

Or check the image interactively:

```bash
docker run --rm gcr.io/instruqt/cloud-client which fail-message
```

### set-status usage

`set-status` is less commonly used in check scripts but can be used to set a custom status message displayed in the Instruqt UI during long-running operations:

```bash
set-status "Validating database schema..."
```

The same image-dependency rules apply.

## What to Watch For

- Bare `fail-message` calls on Docker Hub base images — these will produce `command not found` errors.
- Check scripts that work on one sandbox image but fail on another because of helper availability.
- Custom images that were built without testing whether helpers are installed.
- `set-status` calls in check scripts running on vanilla containers.
- Scripts that pipe into `fail-message` — the helper reads arguments, not stdin.

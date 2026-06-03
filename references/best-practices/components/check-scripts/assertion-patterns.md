# Assertion Patterns

Quality of the assertions used in check scripts to validate learner progress.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Single monolithic check — one pass/fail for the entire challenge with no indication of what went wrong. |
| 2 | Below Standard | Multiple checks exist but are tightly coupled — a failure in one causes misleading failures in others. |
| 3 | Adequate | Independent assertions that each check one thing, but fail-messages are generic or missing for some. |
| 4 | Good | Independent assertions with specific fail-messages, each testable in isolation. Production baseline. |
| 5 | Excellent | Assertions form a diagnostic sequence that pinpoints the exact issue — ordered from prerequisites to specifics, so the first failure is always the most actionable. |

## Guidance

Each assertion should validate exactly one condition and produce exactly one fail-message on failure. Assertions must not depend on each other — a failure in assertion 2 should not cause assertion 3 to fail with a confusing message.

### Good — independent assertions in diagnostic order

```bash
#!/bin/bash

# Check that nginx is installed
if ! command -v nginx &>/dev/null; then
  fail-message "nginx is not installed. Run: apt-get install -y nginx"
  exit 1
fi

# Check that nginx is running
if ! pgrep -x nginx &>/dev/null; then
  fail-message "nginx is installed but not running. Run: systemctl start nginx"
  exit 1
fi

# Check that the config has the correct server_name
if ! grep -q 'server_name lab.example.com;' /etc/nginx/sites-enabled/default; then
  fail-message "The server_name directive is not set to lab.example.com in /etc/nginx/sites-enabled/default."
  exit 1
fi

# Check that the site responds correctly
STATUS=$(curl -s -o /dev/null -w '%{http_code}' --max-time 5 http://localhost/ 2>/dev/null)
if [[ "$STATUS" != "200" ]]; then
  fail-message "Expected HTTP 200 from http://localhost/ but got $STATUS. Check your nginx configuration and restart the service."
  exit 1
fi
```

The ordering matters: installed -> running -> configured -> responding. If nginx is not installed, the learner sees that message first rather than a confusing "curl failed" message.

### Bad — coupled assertions with cascading failures

```bash
#!/bin/bash

OUTPUT=$(curl -s http://localhost/)
if [[ -z "$OUTPUT" ]]; then
  fail-message "Something went wrong"
  exit 1
fi

if ! echo "$OUTPUT" | grep -q "Welcome"; then
  fail-message "Page content is wrong"
  exit 1
fi
```

If nginx is not installed, the learner sees "Something went wrong" — useless. The check does not distinguish between "not installed", "not running", and "wrong content".

### Bad — monolithic check

```bash
#!/bin/bash

command -v nginx &>/dev/null && pgrep -x nginx &>/dev/null && curl -s http://localhost/ | grep -q "Welcome"
if [[ $? -ne 0 ]]; then
  fail-message "The challenge is not complete."
  exit 1
fi
```

One compound condition, one generic message. The learner has no idea which step failed.

## What to Watch For

- Assertions that pipe into each other (output of one feeds input of next) — these are coupled by definition.
- Missing fail-message on any exit path — Instruqt shows a generic error if no fail-message is set.
- Assertions that check a downstream effect without first checking the prerequisite (e.g., checking HTTP response without first checking if the service is running).
- Assertions that silently succeed when the tool they depend on is missing (e.g., `jq` not installed causes an empty-string comparison to pass).

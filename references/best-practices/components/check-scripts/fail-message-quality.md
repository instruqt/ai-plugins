# Fail-Message Quality

Quality of the text shown to learners when a check assertion fails.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No fail-message calls — Instruqt shows "Something went wrong" with no useful information. |
| 2 | Below Standard | Fail-messages exist but are vague ("Check failed", "Not correct", "Try again"). |
| 3 | Adequate | Messages identify what is wrong but not how to fix it ("The file /etc/app.conf does not exist"). |
| 4 | Good | Every fail-message tells the learner exactly what to do — specific command, specific value, specific file. Production baseline. |
| 5 | Excellent | Messages include remediation steps AND context about why the check exists, helping the learner understand the concept, not just pass the gate. |

## Guidance

A fail-message is the primary feedback mechanism for learners. It fires when they click Check and something is wrong. The message must be actionable — the learner should be able to read it, go back to the terminal, and know exactly what to do.

### Good (score 5) — actionable with context

```bash
if ! grep -q '^net.ipv4.ip_forward = 1' /etc/sysctl.conf; then
  fail-message "IP forwarding is not enabled in /etc/sysctl.conf. Add the line 'net.ipv4.ip_forward = 1' to the file and run 'sysctl -p' to apply it. IP forwarding is required for the container network to route traffic between pods."
  exit 1
fi
```

### Good (score 4) — actionable

```bash
if ! grep -q '^net.ipv4.ip_forward = 1' /etc/sysctl.conf; then
  fail-message "IP forwarding is not enabled. Add 'net.ipv4.ip_forward = 1' to /etc/sysctl.conf and run 'sysctl -p'."
  exit 1
fi
```

### Bad (score 2) — vague

```bash
if ! grep -q '^net.ipv4.ip_forward = 1' /etc/sysctl.conf; then
  fail-message "IP forwarding is not configured correctly."
  exit 1
fi
```

### Bad (score 1) — missing

```bash
if ! grep -q '^net.ipv4.ip_forward = 1' /etc/sysctl.conf; then
  exit 1
fi
```

### Message content guidelines

- Start with what is wrong, then say what to do.
- Use the exact filename, command, or value the learner needs.
- Do not blame the learner ("You forgot to..."). State the condition ("The file does not contain...").
- Keep messages under ~200 characters when possible — they render in a toast-style notification.
- If the check validates a specific value, include the expected value in the message.
- Avoid interpolating large variable contents into messages — they can produce unreadable output.

### Including dynamic context

When useful, include the actual value in the message so the learner can see what they produced vs. what was expected:

```bash
ROW_COUNT=$(psql -U postgres -d lab -tAc "SELECT count(*) FROM users;" 2>/dev/null)
if [[ "$ROW_COUNT" != "3" ]]; then
  fail-message "Expected 3 rows in the users table but found ${ROW_COUNT:-0}. Insert the missing rows as described in the assignment."
  exit 1
fi
```

## What to Watch For

- Exit without fail-message — Instruqt shows a generic error with no learner-facing information.
- Messages that reference internal paths or tools the learner cannot access.
- Messages that assume knowledge not covered in the assignment text.
- Messages that say "check failed" without saying what to check.
- Messages with shell variable interpolation that could expand to empty string — use `${VAR:-default}` patterns.

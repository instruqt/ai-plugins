# Lolcat in Lifecycle Scripts

Using `lolcat`, `cowsay`, `ponysay`, or similar tools that produce ANSI escape sequences in lifecycle script output.

## Why It's Bad

ANSI escape sequences (color codes, cursor movement) corrupt the Instruqt debug log. The log becomes unreadable, filled with raw escape characters like `\e[38;5;196m`. This makes debugging track failures significantly harder because the output that should help diagnose issues is garbled.

## What to Do Instead

Use plain text output in lifecycle scripts (setup, check, solve, cleanup). Reserve ANSI-colored or novelty output tools for terminal tab `cmd:` scripts where the output goes directly to a terminal emulator that can render escape sequences.

## Examples

Bad -- lolcat in a setup script:

```bash
# setup-sandbox
echo "Setting up the lab environment..." | lolcat
cowsay "Welcome to the lab!"
```

Good -- plain text in lifecycle scripts:

```bash
# setup-sandbox
echo "Setting up the lab environment..."
echo "Welcome to the lab!"
```

Acceptable -- lolcat in a terminal tab command where a terminal renders it:

```yaml
tabs:
  - title: Terminal
    type: terminal
    cmd: echo "Welcome!" | lolcat
```

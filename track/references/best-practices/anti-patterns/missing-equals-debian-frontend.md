# Missing Equals in DEBIAN_FRONTEND

Writing `export DEBIAN_FRONTEND noninteractive` with a missing `=` sign.

## Why It's Bad

Without the `=`, bash interprets the entire string `DEBIAN_FRONTEND noninteractive` as a variable name (which is invalid but silently ignored in some contexts). The variable never gets set, so `apt-get install` runs in interactive mode. The symptom is that the setup script freezes waiting for user input on a dpkg configuration prompt, and the track times out.

## What to Do Instead

Always include the `=` sign: `export DEBIAN_FRONTEND=noninteractive`.

## Examples

Bad:

```bash
export DEBIAN_FRONTEND noninteractive
apt-get install -y mysql-server
# Freezes on "Configuring mysql-server" prompt
```

Good:

```bash
export DEBIAN_FRONTEND=noninteractive
apt-get install -y mysql-server
```

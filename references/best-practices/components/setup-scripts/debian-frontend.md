# DEBIAN_FRONTEND and Package Management

Evaluates whether setup scripts correctly suppress interactive prompts during package installation, preventing hangs that cause setup timeouts.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No DEBIAN_FRONTEND set; apt-get commands hang on interactive prompts and setup times out |
| 2 | Below Standard | DEBIAN_FRONTEND is set for some but not all apt-get invocations, or uses incorrect syntax (e.g., missing the equals sign) |
| 3 | Adequate | DEBIAN_FRONTEND=noninteractive is set but only as an inline prefix on individual commands rather than exported; apt used instead of apt-get |
| 4 | Good | export DEBIAN_FRONTEND=noninteractive at the top of script; all package operations use apt-get with -y flag (production baseline) |
| 5 | Excellent | Also sets NEEDRESTART_MODE=a for Ubuntu 22+ hosts, uses -qq for reduced output, and handles dpkg lock contention with a retry loop |

## Guidance

Any setup script that installs packages must suppress interactive prompts. The `apt-get` command is preferred over `apt` in scripts because `apt` is designed for interactive terminal use and may produce warnings or behave differently in non-interactive contexts.

Good -- correct export at the top of the script:

```bash
#!/bin/bash
export DEBIAN_FRONTEND=noninteractive

apt-get update -qq
apt-get install -y -qq curl jq unzip
```

Good -- full production setup with needrestart and lock handling:

```bash
#!/bin/bash
export DEBIAN_FRONTEND=noninteractive
export NEEDRESTART_MODE=a

wait_for_dpkg() {
  while fuser /var/lib/dpkg/lock-frontend > /dev/null 2>&1; do
    sleep 2
  done
}

wait_for_dpkg
apt-get update -qq
apt-get install -y -qq curl jq unzip
```

Bad -- missing export keyword (variable not inherited by child processes):

```bash
#!/bin/bash
DEBIAN_FRONTEND noninteractive  # Syntax error, does nothing
apt-get install -y curl
```

Bad -- using apt instead of apt-get:

```bash
#!/bin/bash
export DEBIAN_FRONTEND=noninteractive
apt install -y curl  # apt is for interactive use
```

Bad -- inline prefix only (doesn't cover all commands):

```bash
#!/bin/bash
DEBIAN_FRONTEND=noninteractive apt-get install -y curl
apt-get install -y jq  # This one has no DEBIAN_FRONTEND
```

Bad -- missing -y flag (blocks waiting for confirmation):

```bash
#!/bin/bash
export DEBIAN_FRONTEND=noninteractive
apt-get install curl  # Blocks waiting for Y/n
```

## What to Watch For

- The export must use an equals sign: `export DEBIAN_FRONTEND=noninteractive` not `export DEBIAN_FRONTEND noninteractive`
- Every apt-get install must include `-y` to auto-confirm
- On Ubuntu 22.04+ images, unattended upgrades and needrestart can trigger interactive prompts even with DEBIAN_FRONTEND set; `NEEDRESTART_MODE=a` suppresses needrestart's kernel-check dialog
- The dpkg lock file may be held by unattended-upgrades on fresh VMs; a wait loop prevents failures
- Using `apt` instead of `apt-get` in non-interactive scripts can produce "WARNING: apt does not have a stable CLI interface" messages
- The `-qq` flag suppresses progress output, keeping setup logs clean

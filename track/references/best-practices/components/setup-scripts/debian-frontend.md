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

### debconf-set-selections for packages with required prompts

Some packages (mysql-server, krb5-config, iptables-persistent, postfix) require answers to configuration prompts that `DEBIAN_FRONTEND=noninteractive` alone cannot suppress. Pre-seed the answers with `debconf-set-selections`:

```bash
#!/bin/bash
export DEBIAN_FRONTEND=noninteractive

# Pre-seed MySQL root password
echo "mysql-server mysql-server/root_password password instruqt" | debconf-set-selections
echo "mysql-server mysql-server/root_password_again password instruqt" | debconf-set-selections
apt-get install -y -qq mysql-server

# Pre-seed iptables-persistent to save current rules
echo "iptables-persistent iptables-persistent/autosave_v4 boolean true" | debconf-set-selections
echo "iptables-persistent iptables-persistent/autosave_v6 boolean true" | debconf-set-selections
apt-get install -y -qq iptables-persistent

# Pre-seed Kerberos default realm
echo "krb5-config krb5-config/default_realm string LAB.EXAMPLE.COM" | debconf-set-selections
apt-get install -y -qq krb5-user
```

Without pre-seeding, these packages either hang waiting for input (causing a timeout) or install with incorrect defaults.

### apt-get vs apt

Always use `apt-get` over `apt` in non-interactive scripts. `apt` is designed for interactive terminal use and may produce warnings or behave unpredictably in scripts:

```bash
# Good — apt-get is the scripting interface
apt-get update -qq
apt-get install -y -qq curl jq

# Bad — apt is for interactive use
apt update        # "WARNING: apt does not have a stable CLI interface"
apt install -y curl
```

## What to Watch For

- The export must use an equals sign: `export DEBIAN_FRONTEND=noninteractive` not `export DEBIAN_FRONTEND noninteractive`
- Every apt-get install must include `-y` to auto-confirm
- On Ubuntu 22.04+ images, unattended upgrades and needrestart can trigger interactive prompts even with DEBIAN_FRONTEND set; `NEEDRESTART_MODE=a` suppresses needrestart's kernel-check dialog
- The dpkg lock file may be held by unattended-upgrades on fresh VMs; a wait loop prevents failures
- Using `apt` instead of `apt-get` in non-interactive scripts can produce "WARNING: apt does not have a stable CLI interface" messages
- The `-qq` flag suppresses progress output, keeping setup logs clean
- Packages with interactive prompts (mysql-server, krb5-config, iptables-persistent, postfix) need `debconf-set-selections` before installation — `DEBIAN_FRONTEND=noninteractive` only suppresses the UI, it does not provide answers

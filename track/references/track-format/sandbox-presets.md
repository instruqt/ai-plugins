# Sandbox Presets

A sandbox preset provisions a pre-built environment as an alternative to defining infrastructure in `config.yml`. When using a preset, no `config.yml` or `track_scripts/` directory is needed -- the environment comes ready to use.

## Structure

Presets are selected in the Instruqt web UI at the track level. The track repository contains only challenge directories with assignments and (optionally) per-challenge lifecycle scripts.

## Preset Families

### SUSE / Cloud Native

| Preset | Hostname(s) | Description |
|--------|-------------|-------------|
| `sles-15-cloud-native` | `console` | SLES 15 with az, kubectl, helm pre-installed |

### Elastic

| Preset | Hostname(s) | Description |
|--------|-------------|-------------|
| `eck-8-15-0-genai-wave3` | `kubernetes-vm`, `lab-host-1` | ECK environment for GenAI workshops |
| `managed-vm-elastic-highmem-*` | `kubernetes-vm`, `lab-host-1` | ECK pre-deployed, high memory. Zero `track_scripts` needed. |
| `managed-elastic-9-x-0` | `kubernetes-vm`, `lab-host-1` | Lighter-weight preset for ES\|QL / serverless workshops |
| `managed-vscode-elastic-*` | `kubernetes-vm`, `lab-host-1` | VS Code environment alongside Elastic stack |

### Isovalent / Cilium

| Preset | Hostname(s) | Description |
|--------|-------------|-------------|
| `highcpu8-nokind` | `server` | High-CPU node without Kind |
| `highcpu8-2w-kpr` | `server` | High-CPU node with KPR |

### Temporal

| Preset | Hostname(s) | Description |
|--------|-------------|-------------|
| `temporal` | `temporal1` | Typical tabs: VS Code on 8443, Temporal UI on 8080 |

### Other

| Preset | Hostname(s) | Description |
|--------|-------------|-------------|
| `dev-platform-node-gen2` | -- | Node-capable dev environment |
| `aws-bootcamp-preset` | `cloud-client`, `vensim-host` | Illumio AWS bootcamp |
| `cockroach-university-self-paced-preset` | -- | CockroachDB University self-paced labs |

## Hostname Cheat Sheet

The hostname to use in lifecycle scripts and tab definitions depends on the preset:

| Preset Pattern | Hostname(s) |
|----------------|-------------|
| `eck-*`, `managed-*-elastic-*` | `kubernetes-vm` + `lab-host-1` |
| `sles-*` | `console` |
| `highcpu8-*` | `server` |
| `temporal` | `temporal1` |

Always check the preset documentation for the authoritative hostname list -- different presets within the same family may vary.

## Examples

### Minimal Track with Preset

A track using `managed-vm-elastic-highmem-*` needs no `config.yml` and no `track_scripts/`. The directory contains only challenge folders:

```
my-elastic-track/
  01-first-challenge/
    assignment.md
    setup-kubernetes-vm    # optional per-challenge setup
  02-second-challenge/
    assignment.md
    check-kubernetes-vm
    solve-kubernetes-vm
```

### Per-Challenge Script Targeting Preset Hostname

```bash
#!/bin/bash
set -euxo pipefail

# Preset: eck-8-15-0-genai-wave3
# Hostname: kubernetes-vm
kubectl get pods -n elastic-system
```

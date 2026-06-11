# Sandbox Presets

A sandbox preset provisions a pre-built environment as an alternative to defining infrastructure in `config.yml`. When using a preset, no `config.yml` or `track_scripts/` directory is needed -- the environment comes ready to use.

## Structure

Presets are selected in the Instruqt web UI at the track level. The track repository contains only challenge directories with assignments and (optionally) per-challenge lifecycle scripts.

## Preset Families

### Vendor A (Cloud Native)

| Preset | Hostname(s) | Description |
|--------|-------------|-------------|
| `vendor-a-cloud-native` | `console` | Linux with az, kubectl, helm pre-installed |

### Vendor B (Search & Observability)

| Preset | Hostname(s) | Description |
|--------|-------------|-------------|
| `vendor-b-genai-wave3` | `kubernetes-vm`, `lab-host-1` | Kubernetes operator environment for GenAI workshops |
| `vendor-b-highmem-*` | `kubernetes-vm`, `lab-host-1` | Operator pre-deployed, high memory. Zero `track_scripts` needed. |
| `vendor-b-query-9-x-0` | `kubernetes-vm`, `lab-host-1` | Lighter-weight preset for query / serverless workshops |
| `vendor-b-vscode-*` | `kubernetes-vm`, `lab-host-1` | VS Code environment alongside search stack |

### Vendor C (Networking)

| Preset | Hostname(s) | Description |
|--------|-------------|-------------|
| `vendor-c-highcpu-nokind` | `server` | High-CPU node without Kind |
| `vendor-c-highcpu-2w-kpr` | `server` | High-CPU node with KPR |

### Vendor D (Workflow Engine)

| Preset | Hostname(s) | Description |
|--------|-------------|-------------|
| `vendor-d-workflow` | `workflow1` | Typical tabs: VS Code on 8443, workflow UI on 8080 |

### Other

| Preset | Hostname(s) | Description |
|--------|-------------|-------------|
| `dev-platform-node-gen2` | -- | Node-capable dev environment |
| `vendor-e-aws-bootcamp` | `cloud-client`, `vensim-host` | Vendor E AWS bootcamp |
| `vendor-f-university-self-paced` | -- | Vendor F university self-paced labs |

## Hostname Cheat Sheet

The hostname to use in lifecycle scripts and tab definitions depends on the preset:

| Preset Pattern | Hostname(s) |
|----------------|-------------|
| `vendor-b-*` | `kubernetes-vm` + `lab-host-1` |
| `vendor-a-*` | `console` |
| `vendor-c-*` | `server` |
| `vendor-d-*` | `workflow1` |

Always check the preset documentation for the authoritative hostname list -- different presets within the same family may vary.

## Examples

### Minimal Track with Preset

A track using `vendor-b-highmem-*` needs no `config.yml` and no `track_scripts/`. The directory contains only challenge folders:

```
my-preset-track/
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

# Preset: vendor-b-genai-wave3
# Hostname: kubernetes-vm
kubectl get pods -n operator-system
```

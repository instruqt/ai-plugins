# Challenge: [Title]

## Context

[Why this challenge exists in the track — what it connects to before and after]

## Prior State

[What the environment looks like when the learner arrives at this challenge]
- Services running: [list]
- Files present: [list]
- State from previous challenges: [description]

## Prerequisites

Capabilities that must be **functional** (not merely installed) when the learner arrives at this challenge. Each row is something the assignment depends on, plus the command that *proves* it works. "Provided by" ties the capability to where it is established — track setup, an earlier challenge, or this challenge's setup. The setup script that provides each capability must verify it (see the verification tail in `write-scripts`); the validator checks every row here has a matching assertion.

State each capability as a functional outcome, not a package name — "kubectl reaches the cluster, context X" not "kubectl installed"; "interpreter can import the SDK" not "pip install ran". The Verify command must distinguish *present* from *working* (e.g. `gcloud auth list` proves auth, not just that `gcloud` is on PATH).

| Capability (functional at challenge start) | Host | Verify command | Provided by |
|--------------------------------------------|------|----------------|-------------|
| [e.g. gcloud authenticated as the workshop service account] | [host] | [e.g. `gcloud auth list --filter=status:ACTIVE --format='value(account)'`] | [track setup / NN-slug / this challenge] |
| [e.g. kubectl reaches the cluster on context "workshop"] | [host] | [e.g. `kubectl cluster-info`] | [track setup] |
| [e.g. interpreter can import the SDK the assignment uses] | [host] | [e.g. `<interpreter> -c "<import smoke test>"`] | [this challenge] |

## Assignment Outline

1. [Step 1 — what the learner reads and does]
2. [Step 2]
3. [Step N]

## Tabs

| Title | Type | Target | Notes |
|-------|------|--------|-------|
| [Tab 1] | terminal | hostname: [host] | [workdir, cmd if applicable] |
| [Tab 2] | service | hostname: [host], port: [port] | [path if applicable] |

## Scripts

### Setup (`setup-[hostname]`)

- [What to install/configure before the learner starts]
- [Services to start]
- [Files to create]
- Verification tail: assert each Prerequisites row this script provides is functional (install → verify), failing loudly if any is not

### Check (`check-[hostname]`)

- [Assertion 1: what to verify + fail message]
- [Assertion 2: what to verify + fail message]

### Solve (`solve-[hostname]`)

- [Command 1: matches assignment step 1]
- [Command 2: matches assignment step 2]

### Cleanup (`cleanup-[hostname]`)

- [Resources to tear down, if any]
- [State to reset, if any]

## Infrastructure Changes

[Any additions to config.yml needed for this challenge]
- New ports to expose: [list]
- New services to add: [list]
- Environment variables needed: [list]

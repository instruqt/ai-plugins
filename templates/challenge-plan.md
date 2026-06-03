# Challenge: [Title]

## Context

[Why this challenge exists in the track — what it connects to before and after]

## Prior State

[What the environment looks like when the learner arrives at this challenge]
- Services running: [list]
- Files present: [list]
- State from previous challenges: [description]

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

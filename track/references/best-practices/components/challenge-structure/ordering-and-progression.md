# Ordering and Progression

Evaluates whether challenges are ordered to create a logical learning flow with progressive difficulty and no concept jumps.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Challenge order is random or incoherent; advanced tasks appear before foundational concepts; learner is lost by challenge 2 |
| 2 | Below Standard | Some logical flow exists but there are concept jumps -- a challenge assumes knowledge that was not covered in prior challenges |
| 3 | Adequate | Challenges follow a logical sequence and concepts generally build on each other, but the progression is uneven (difficulty spikes or redundant steps) |
| 4 | Good | Logical progression with no concept jumps; each challenge builds on prior knowledge; difficulty ramps steadily (production baseline) |
| 5 | Excellent | Each challenge adds exactly one new concept while reinforcing prior ones; progression follows observation then guided then independent pattern |

## Guidance

Challenge ordering should follow a deliberate pedagogical progression. The learner should never encounter a concept or tool for the first time without introduction, and should never feel that a challenge is disconnected from what came before.

### The Three-Phase Pattern

A well-structured track follows this arc:

1. **Observation** -- Early challenges where the learner explores a pre-built environment, reads output, and builds mental models. Low risk, high context.
2. **Guided** -- Middle challenges where the learner performs actions with step-by-step instructions. The assignment tells them exactly what to do.
3. **Independent** -- Later challenges where the learner applies learned concepts with less hand-holding. Instructions describe the goal, not the steps.

Good -- progressive three-phase track:

```
01-explore-the-cluster/        # Observation: look at what's running
02-understand-the-manifest/     # Observation: read and understand config
03-deploy-the-application/      # Guided: follow steps to deploy
04-configure-networking/         # Guided: set up ingress with instructions
05-scale-the-deployment/         # Independent: goal given, fewer hints
06-troubleshoot-an-issue/        # Independent: diagnose and fix on their own
```

Bad -- advanced task before foundation:

```
01-deploy-to-kubernetes/        # Assumes K8s knowledge not yet taught
02-what-is-kubernetes/          # Foundation comes too late
03-configure-networking/
```

Bad -- no progression (all guided):

```
01-do-step-one/
02-do-step-two/
03-do-step-three/
04-do-step-four/
# Every challenge is identical in structure and difficulty
```

### Concept Dependencies

Map out which concepts each challenge introduces and which it depends on. No challenge should depend on a concept that was not introduced in a prior challenge.

Good -- clear dependency chain:

```
01-create-a-vault-server       # Introduces: Vault server, seal/unseal
02-authenticate-to-vault       # Depends on: running server. Introduces: auth methods
03-write-secrets               # Depends on: authentication. Introduces: KV engine
04-create-policies             # Depends on: secrets. Introduces: ACL policies
05-use-dynamic-credentials     # Depends on: policies. Introduces: dynamic secrets
```

Bad -- broken dependency:

```
01-create-a-vault-server       # Introduces: Vault server
02-use-dynamic-credentials     # Depends on: policies, KV engine (not yet taught!)
03-write-secrets               # Too late -- challenge 2 already needed this
```

### One New Concept Per Challenge

The ideal challenge introduces exactly one new concept while exercising previously learned ones. This keeps cognitive load manageable and provides natural reinforcement.

Good -- one concept per challenge:

```
01-introduction          # New: the product, the UI
02-create-a-project      # New: project creation. Reinforces: UI navigation
03-add-team-members      # New: team management. Reinforces: project context
04-configure-permissions # New: RBAC. Reinforces: teams, projects
```

Bad -- too many new concepts at once:

```
01-introduction
02-create-project-add-teams-configure-rbac  # Three new concepts in one challenge
03-deploy-and-monitor                        # Two more new concepts
```

## What to Watch For

- A challenge that references commands, tools, or concepts not introduced in any prior challenge
- Difficulty that spikes suddenly (challenge 2 is trivial, challenge 3 is expert-level)
- Challenges that could be reordered without breaking anything -- this suggests they are independent rather than building on each other
- "Kitchen sink" challenges that pack multiple new concepts into one step
- Missing an introductory/exploration challenge -- learners benefit from seeing the environment before changing it
- Final challenge that introduces new concepts instead of synthesizing prior learning

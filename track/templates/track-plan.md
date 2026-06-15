# Track: [Title]

## Intent

[What the learner should be able to do after completing this track]

## Products Covered

- [Product 1]
- [Product 2]

## Learning Objectives

1. [Objective 1]
2. [Objective 2]
3. [Objective 3]

## Target Audience

- **Role:** [e.g., DevOps engineer, platform engineer, developer]
- **Experience level:** [e.g., Beginner with containers, intermediate with Kubernetes]
- **Prerequisites:** [What the learner should already know]

## Sandbox Requirements

- **Infrastructure pattern:** [e.g., single-container, single-vm, vm-plus-cloud-account]
- **Hosts:** [List of containers/VMs needed]
- **Cloud accounts:** [If applicable]
- **Estimated startup time:** [e.g., ~30s for containers, ~2min for VMs]
- **Special requirements:** [e.g., nested virtualization, GPU, specific machine type]

## Track-Level Prerequisites

Capabilities installed **once** by track-level setup (`track_scripts/setup-<host>`) and relied on across challenges — base CLIs, runtimes, authenticated cloud access, shared services. State each as a functional outcome with the command that proves it works; challenge plans reference these as "Provided by: track setup" rather than re-declaring them. Track setup must verify each one (see the verification tail in `write-scripts`).

| Capability (functional after track setup) | Host | Verify command |
|--------------------------------------------|------|----------------|
| [e.g. gcloud authenticated as the workshop service account] | [host] | [e.g. `gcloud auth list --filter=status:ACTIVE --format='value(account)'`] |
| [e.g. kubectl reaches the cluster on context "workshop"] | [host] | [e.g. `kubectl cluster-info`] |

## Track Configuration

- **Duration:** [e.g., ~45 min]
- **Difficulty:** [Basic / Intermediate / Advanced / Basic → Advanced (progressive)]
- **Pausable:** No (default — enable only if the user explicitly requests pause/resume; add TTL if so)
- **Show timer:** Yes (default — hide only for presenter-paced or open-ended tracks)
- **Hot start:** [Recommended / Not needed]

## Challenge Roadmap

### Challenge 1: [Title]

| Field | Value |
|-------|-------|
| **Slug** | [01-slug-name] |
| **Goal** | [What the learner accomplishes] |
| **Concepts** | [Key concepts introduced] |
| **Time** | [Estimated minutes] |
| **Difficulty** | [basic / intermediate / advanced] |

### Challenge 2: [Title]

| Field | Value |
|-------|-------|
| **Slug** | [02-slug-name] |
| **Goal** | [What the learner accomplishes] |
| **Concepts** | [Key concepts introduced] |
| **Time** | [Estimated minutes] |
| **Difficulty** | [basic / intermediate / advanced] |

### Challenge N: [Title]

| Field | Value |
|-------|-------|
| **Slug** | [NN-slug-name] |
| **Goal** | [What the learner accomplishes] |
| **Concepts** | [Key concepts introduced] |
| **Time** | [Estimated minutes] |
| **Difficulty** | [basic / intermediate / advanced] |

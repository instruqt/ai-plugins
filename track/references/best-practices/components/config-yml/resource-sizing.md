# Resource Sizing

Evaluates whether VM and container resources are appropriately sized for the workload, balancing cost efficiency with learner experience.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No sizing specified (bare defaults), or wildly oversized resources (n2-highcpu-32 for a simple CLI exercise) |
| 2 | Below Standard | Sizing specified but uses raw memory/cpus instead of machine_type, or sizing clearly mismatched to workload |
| 3 | Adequate | machine_type used but not well-matched to workload -- e.g., n1-standard-1 for a Kubernetes cluster or n1-standard-4 for a simple script exercise |
| 4 | Good | Sizing appropriate for the workload; machine_type used consistently; containers have explicit memory where needed (production baseline) |
| 5 | Excellent | Sizing optimized for cost vs performance -- smallest machine_type that runs the workload comfortably, with documented rationale for non-obvious choices |

## Guidance

Always use `machine_type` instead of specifying raw `memory` and `cpus` values. Machine types map to well-known GCP instance profiles and ensure consistent, reproducible environments.

Common machine_type selections:

- **n1-standard-1** -- Minimal VM for lightweight CLI tools, file editing, simple scripts
- **n1-standard-2** -- General purpose; sufficient for most single-service workloads (Docker, single-node apps, basic Terraform)
- **n1-standard-4** -- Multi-service workloads, local Kubernetes (k3s, minikube), medium databases
- **n2-highcpu-8** -- CPU-intensive builds, compilation-heavy tasks, multi-container orchestration
- **n2-highcpu-16** -- Large Kubernetes clusters (kind with multiple nodes), heavy compilation
- **n2-highcpu-32** -- Nested virtualization, OpenShift, multi-node Kubernetes with substantial workloads

For containers, the default memory allocation is usually sufficient for lightweight images. Override only when the container runs memory-intensive processes.

Good -- machine_type matched to workload:

```yaml
virtualmachines:
- name: workstation
  machine_type: n1-standard-2
  image: ubuntu-22-04
```

Good -- larger sizing with rationale:

```yaml
# Nested virt requires n2-highcpu-32 for acceptable performance
virtualmachines:
- name: openshift-node
  machine_type: n2-highcpu-32
  image: rhel-9
```

Good -- container with explicit memory for heavy workload:

```yaml
containers:
- name: database
  image: postgres:16
  memory: 2048
```

Bad -- raw memory/cpus instead of machine_type:

```yaml
virtualmachines:
- name: workstation
  memory: 4096
  cpus: 2
  image: ubuntu-22-04
```

Bad -- oversized for the workload:

```yaml
# Only running a simple Python script
virtualmachines:
- name: workstation
  machine_type: n2-highcpu-16
  image: ubuntu-22-04
```

## What to Watch For

- Raw `memory` and `cpus` fields instead of `machine_type` -- these are less predictable and harder to maintain
- Oversized VMs that waste cloud resources and increase track costs
- Undersized VMs that cause timeouts, OOM kills, or sluggish learner experience
- Containers without explicit memory when running databases, build tools, or JVM-based applications
- Nested virtualization workloads (OpenShift, KVM) that require n2-highcpu-32 -- smaller types will fail silently or hang

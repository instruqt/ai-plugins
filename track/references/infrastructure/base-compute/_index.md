# Base Compute Reference Index

Base compute patterns define the fundamental sandbox hosts for a track. Every track uses exactly one base compute pattern.

## Container-Based

- `single-container.md` — One Docker container for CLI-only workshops
- `multi-container-multi-tier.md` — Distinct services composed together (web + db + cache)
- `multi-container-clustered.md` — N instances of the same service forming a cluster
- `multi-container-sidecar.md` — Helper container tightly coupled to a primary container

## VM-Based

- `single-vm.md` — One VM with native package installs and systemd services (no Docker)
- `single-vm-docker.md` — One VM running one or a few Docker containers
- `single-vm-docker-compose.md` — One VM with a multi-service Docker Compose stack
- `single-vm-kind-k3s.md` — One VM running a Kind or k3s Kubernetes cluster
- `single-vm-fat-stack.md` — One large VM running a self-contained multi-service deployment

## Multi-Host

- `multi-vm.md` — Multiple VMs for distributed systems, clusters, leader/worker topologies
- `container-plus-vm.md` — Hybrid: lightweight CLI container as workstation, VM runs heavier services

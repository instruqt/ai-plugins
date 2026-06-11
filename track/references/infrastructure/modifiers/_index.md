# Modifiers Reference Index

Modifiers are cross-cutting capabilities that layer onto base compute patterns. They describe how to add a specific technology or feature to a track's infrastructure.

## VM-Only Modifiers

These require Docker socket access, which is only available on VMs (not containers).

- `docker-on-vm.md` — Docker daemon on a VM for running containers alongside systemd services
- `docker-compose-on-vm.md` — Docker Compose on a VM for orchestrating multi-service stacks
- `kind-k3s-on-vm.md` — Kind or k3s on a VM for running Kubernetes clusters
- `containerlab-on-vm.md` — Containerlab on a VM for virtual network topologies (BGP, OSPF, etc.)

## General Modifiers

These apply to any base compute pattern (container or VM).

- `custom-image-packer.md` — Custom VM images built with Packer for fast startup with pre-baked dependencies
- `custom-image-docker.md` — Custom Docker container images with pre-installed tools and config
- `ssl-certificates.md` — SSL certificate provisioning for VM hostnames (HTTPS access)
- `private-repo-access.md` — Accessing private Git repositories from sandbox hosts via secrets
- `account-pool-checkout.md` — Reserving external accounts or tenants from a pool per learner session

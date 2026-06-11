# Infrastructure Reference Index

## Base Compute

- `base-compute/single-container.md` — Single container: CLI-only workshops, no VM-level features needed
- `base-compute/multi-container-multi-tier.md` — Multi-container multi-tier: distinct services composed together (web + db + cache)
- `base-compute/multi-container-clustered.md` — Multi-container clustered: N instances of the same service forming a cluster with discovery and health-gating
- `base-compute/multi-container-sidecar.md` — Multi-container sidecar: helper container tightly coupled to a primary, sharing network/volumes
- `base-compute/single-vm.md` — Single VM (bare): systemd services, native package installs, no Docker
- `base-compute/single-vm-docker.md` — Single VM + Docker: one or few containers on a VM, not Compose-worthy
- `base-compute/single-vm-docker-compose.md` — Single VM + Docker Compose: multi-service stack orchestrated by Compose on one VM
- `base-compute/single-vm-kind-k3s.md` — Single VM + Kind/k3s: Kubernetes cluster on a single VM
- `base-compute/single-vm-fat-stack.md` — Single VM fat stack: self-contained multi-service deployment on a large VM (high memory/CPU)
- `base-compute/multi-vm.md` — Multi-VM: distributed systems, leader/worker topologies, replica sets, multi-tier across hosts
- `base-compute/container-plus-vm.md` — Container + VM hybrid: lightweight CLI container as workstation, VM runs heavier services

## Cloud Accounts

- `cloud-accounts/container-plus-aws.md` — Container + AWS: cloud-client container paired with a sandboxed AWS account
- `cloud-accounts/container-plus-azure.md` — Container + Azure: cloud-client container paired with a sandboxed Azure subscription
- `cloud-accounts/container-plus-gcp.md` — Container + GCP: cloud-client container paired with a sandboxed GCP project
- `cloud-accounts/vm-plus-aws.md` — VM + AWS: full VM workstation paired with a sandboxed AWS account
- `cloud-accounts/vm-plus-azure.md` — VM + Azure: full VM workstation paired with a sandboxed Azure subscription
- `cloud-accounts/vm-plus-gcp.md` — VM + GCP: full VM workstation paired with a sandboxed GCP project
- `cloud-accounts/hybrid-plus-cloud.md` — Container + VM + cloud: hybrid compute paired with a cloud account
- `cloud-accounts/multi-cloud.md` — Multi-cloud: multiple cloud providers in a single track

## Managed Kubernetes

- `managed-kubernetes/container-plus-eks.md` — Container + EKS: cloud-client provisions or interacts with AWS EKS
- `managed-kubernetes/container-plus-aks.md` — Container + AKS: cloud-client provisions or interacts with Azure AKS (and optionally ACR)
- `managed-kubernetes/container-plus-gke.md` — Container + GKE: cloud-client provisions or interacts with GCP GKE
- `managed-kubernetes/vm-kind-plus-eks.md` — VM (Kind) + EKS: local Kind cluster paired with a remote EKS cluster
- `managed-kubernetes/vm-kind-plus-aks.md` — VM (Kind) + AKS: local Kind cluster paired with a remote AKS cluster

## Runtime Provisioning

- `runtime-provisioning/terraform-aws-vm.md` — Terraform + AWS VM: container provisions a VM in AWS at setup time
- `runtime-provisioning/terraform-azure-vm.md` — Terraform + Azure VM: container provisions a VM in Azure at setup time
- `runtime-provisioning/terraform-gcp-vm.md` — Terraform + GCP VM: container provisions a VM in GCP at setup time (may use IAP tunnels)

## Learner UX

- `learner-ux/native/terminal-tab.md` — Terminal tab: shell access to a sandbox host, with cmd variant for SSH/REPL
- `learner-ux/native/service-tab.md` — Service tab: iframe proxy to a web UI running inside the sandbox
- `learner-ux/native/editor-tab.md` — Editor tab: in-browser Monaco editor for file editing on a sandbox host
- `learner-ux/native/virtual-browser-sandbox.md` — Virtual browser (sandbox): full Chromium browser pointing at an internal sandbox service
- `learner-ux/native/virtual-browser-external.md` — Virtual browser (external): full Chromium browser pointing at a SaaS or cloud console URL
- `learner-ux/installed/code-server.md` — Code-server: VS Code in browser, requires VM + service tab
- `learner-ux/installed/guacamole-rdp.md` — Guacamole RDP: browser-based GUI desktop via Guacamole + RDP
- `learner-ux/installed/guacamole-vnc.md` — Guacamole VNC: browser-based GUI via Guacamole + VNC protocol
- `learner-ux/installed/kasmvnc.md` — KasmVNC: browser-based GUI application via KasmVNC container
- `learner-ux/installed/novnc.md` — noVNC: lightweight browser-based VNC without Guacamole
- `learner-ux/installed/ttyd.md` — ttyd: browser-streamed terminal for TUI apps (k9s, htop, log tails) via service tab

## Modifiers

- `modifiers/docker-on-vm.md` — Docker on VM: Docker daemon on a virtual machine (VM-only, requires nested_virtualization)
- `modifiers/docker-compose-on-vm.md` — Docker Compose on VM: multi-service orchestration with Compose on a VM (VM-only)
- `modifiers/kind-k3s-on-vm.md` — Kind/k3s on VM: Kubernetes cluster tooling on a VM (VM-only)
- `modifiers/containerlab-on-vm.md` — Containerlab on VM: virtual network topology creation on a VM (VM-only)
- `modifiers/custom-image-packer.md` — Custom VM image (Packer): pre-baked VM with dependencies for fast startup
- `modifiers/custom-image-docker.md` — Custom Docker image: pre-built container image with tools and config
- `modifiers/ssl-certificates.md` — SSL certificates: HTTPS certificate provisioning for VM hostnames
- `modifiers/private-repo-access.md` — Private repo access: cloning private Git repositories via secrets
- `modifiers/account-pool-checkout.md` — Account pool checkout: reserving external accounts/tenants per learner session

## Sandbox Presets

- `sandbox-presets.md` — Sandbox preset: pre-built environment from the Instruqt catalog, no config.yml needed

# Container vs VM

Deciding whether the track's primary compute should be an Instruqt container or a virtual machine.

## The Docker Socket Rule

If the track needs to run Docker commands, it **must** use a VM. Instruqt containers do not have Docker socket access. This single constraint eliminates containers for any track involving:

- `docker run`, `docker build`, `docker compose`
- Kind, k3s, or any Kubernetes tool that creates containers
- Containerlab or any tool that manages Docker networks
- Any nested container execution

## Decision Criteria

### Use a container when ALL of these are true:

- Track is CLI-only (no Docker, no systemd, no kernel features).
- No GUI desktop or graphical application needed.
- Memory requirements are under ~1 GB.
- Startup speed is a priority (containers start faster than VMs).
- `gcr.io/instruqt/shell` or `gcr.io/instruqt/cloud-client:2` has the tools you need (or a custom image does).

### Use a VM when ANY of these are true:

- Track needs Docker, Docker Compose, Kind, k3s, or Containerlab.
- Track needs systemd services (e.g., `systemctl start mongod`).
- Track needs kernel-level features (iptables, cgroups, namespaces, eBPF).
- Track needs a GUI desktop (Guacamole, KasmVNC, noVNC, code-server).
- Track installs software via apt/yum/dnf package managers as part of the exercise.
- Track needs nested virtualization.
- Track needs more than ~1 GB of memory.

## Trade-offs

| Factor | Container | VM |
|--------|-----------|-----|
| Startup time | ~30 seconds | ~2 minutes |
| Docker support | No | Yes (with `nested_virtualization: true`) |
| systemd | No | Yes |
| GUI desktop | No | Yes |
| Memory | 256 MB - 1 GB typical | 2 GB - 128 GB |
| Cost per session | Lower | Higher |
| Package managers | Limited (Alpine/Debian base) | Full OS (apt, yum, dnf) |
| Kernel access | No | Yes |

## Common Mistakes

- **Choosing a container then realizing Docker is needed mid-development.** Check upfront whether any step in the track runs Docker commands. Migrating from container to VM mid-track is painful.
- **Choosing a VM when a container would suffice.** If the track only uses CLI tools (curl, aws, gcloud, terraform, kubectl), a cloud-client container is faster and cheaper. Don't pay for a VM you don't need.
- **Hybrid option overlooked.** If the track needs a heavy service (database, app server) AND a lightweight CLI workstation, consider `container-plus-vm` — the container is the workstation, the VM runs the service. Cheaper than two VMs.

## Quick Reference

```
Does the track need Docker?           --> VM
Does the track need systemd?          --> VM
Does the track need a GUI?            --> VM
Does the track need kernel features?  --> VM
Is it CLI-only with cloud tools?      --> Container (cloud-client)
Is it CLI-only with basic tools?      --> Container (shell)
Need heavy services + light CLI?      --> Container + VM hybrid
```

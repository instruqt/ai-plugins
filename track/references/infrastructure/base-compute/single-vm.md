# Single VM (Bare)

One virtual machine with native package installs and systemd services. No Docker. Use when the track teaches OS-level concepts, installs software via package managers, or runs services managed by systemd.

For VMs that also run Docker containers, see `single-vm-docker.md`, `single-vm-docker-compose.md`, or `single-vm-kind-k3s.md`.

## Config.yml Example

```yaml
version: "3"
virtualmachines:
  - name: workstation
    image: ubuntu-os-cloud/ubuntu-2404-lts-amd64
    machine_type: n1-standard-2
    shell: /bin/bash
    allow_external_ingress:
      - http
      - https
      - high-ports
    provision_ssl_certificate: true
```

### Common Images

| Image | What it provides |
|-------|-----------------|
| `ubuntu-os-cloud/ubuntu-2404-lts-amd64` | Bare Ubuntu 24.04, install what you need in setup |
| `ubuntu-os-cloud/ubuntu-2204-lts` | Ubuntu 22.04 LTS |
| `instruqt-alma-9-base` | AlmaLinux 9 base |

## When to Use

- Track installs software via apt/yum/dnf and manages it with systemd.
- Track teaches Linux administration (storage, networking, users, permissions).
- Track teaches database administration where the DB runs as a native service (MongoDB, PostgreSQL, MySQL).
- Track needs kernel-level features (iptables, cgroups, namespaces) without Docker abstraction.

## When NOT to Use

- Track needs Docker -- use `single-vm-docker.md` or `single-vm-docker-compose.md` instead.
- Track is CLI-only with no OS-level dependencies -- use `single-container.md` for faster startup.
- Track needs multiple hosts -- use `multi-vm.md` or `container-plus-vm.md`.

## Common Pitfalls

- **Long setup times on bare images**. Installing software from package managers in setup scripts can take minutes. For heavy dependency stacks, consider a custom Packer image (see `modifiers/custom-image-packer.md`).
- **Forgetting `allow_external_ingress`**. Without it, service tabs cannot reach the VM. Include `http`, `https`, and `high-ports` for web-facing tracks.
- **Forgetting `provision_ssl_certificate: true`**. Service tabs served over HTTPS will show certificate errors without it.
- **Missing `shell` field**. Without it, terminal tabs may default to an unexpected shell.
- **Package repository freshness**. Run `apt-get update` in setup before installing packages to avoid stale package list errors.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Linux administration (systemd, networking, storage, user management)
- Database installation and administration (MongoDB, PostgreSQL, MySQL as native services)
- Web server configuration (nginx, Apache as systemd services)
- Security fundamentals (iptables, firewalld, SELinux, AppArmor)
- Package management and system configuration tutorials

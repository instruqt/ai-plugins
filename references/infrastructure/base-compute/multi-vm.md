# Multi-VM

Multiple virtual machines for distributed system workshops: cluster setups, multi-tier architectures, leader/worker topologies, or cross-host coordination.

## Config.yml Example

```yaml
version: "3"
virtualmachines:
  - name: control-plane
    image: ubuntu-os-cloud/ubuntu-2404-lts-amd64
    machine_type: n1-standard-4
    shell: /bin/bash
    environment:
      ROLE: server
      WORKER_HOSTNAME: worker
    allow_external_ingress:
      - http
      - https
      - high-ports
    nested_virtualization: true
    provision_ssl_certificate: true

  - name: worker
    image: ubuntu-os-cloud/ubuntu-2404-lts-amd64
    machine_type: n1-standard-2
    shell: /bin/bash
    environment:
      ROLE: agent
      CONTROL_PLANE_HOSTNAME: control-plane
    nested_virtualization: true
```

### Variant: three-node symmetric cluster

```yaml
version: "3"
virtualmachines:
  - name: node-1
    image: ubuntu-os-cloud/ubuntu-2404-lts-amd64
    machine_type: n1-standard-2
    shell: /bin/bash
    environment:
      NODE_ID: "1"
      CLUSTER_NODES: "node-1,node-2,node-3"
    nested_virtualization: true

  - name: node-2
    image: ubuntu-os-cloud/ubuntu-2404-lts-amd64
    machine_type: n1-standard-2
    shell: /bin/bash
    environment:
      NODE_ID: "2"
      CLUSTER_NODES: "node-1,node-2,node-3"
    nested_virtualization: true

  - name: node-3
    image: ubuntu-os-cloud/ubuntu-2404-lts-amd64
    machine_type: n1-standard-2
    shell: /bin/bash
    environment:
      NODE_ID: "3"
      CLUSTER_NODES: "node-1,node-2,node-3"
    nested_virtualization: true
```

## Cross-VM Communication

All VMs in a sandbox share a private network. Each VM's `name` is its hostname, so `ping worker` from `control-plane` works out of the box.

### State sharing strategies

| Method | Best for | How |
|--------|----------|-----|
| **Environment variables** | Static config (hostnames, roles, tokens) | `environment:` block in config.yml |
| **Agent variable** | Single dynamic values, tab interpolation | `agent variable set`, read with `agent variable get` or `[[ Instruqt-Var ]]` |
| **JSON broker sidecar** | Structured payloads consumed by multiple VMs | HTTP server on a port returning JSON, other VMs curl to get credentials/config |
| **SSH / SCP** | File transfer between VMs | Pre-distribute SSH keys in setup scripts |

See `decision-frameworks/agent-variable-vs-json-broker.md` for guidance on choosing between these. Complex tracks often combine all three: env vars for hostnames, agent variables for tab interpolation, JSON broker for structured cross-VM credential passing.

## When to Use

- Track teaches distributed systems that require separate hosts (replication, clustering, federation).
- Leader/worker or primary/replica topologies where each node has a distinct role.
- Network exercises between hosts (firewall rules, routing, service mesh).
- Track needs isolated environments for different roles (admin workstation + production servers).

## When NOT to Use

- Multiple services on one host is sufficient -- use `single-vm-docker-compose.md` instead (cheaper, faster startup).
- Only need a lightweight CLI workstation alongside a VM -- use `container-plus-vm.md` to save resources.

## Common Pitfalls

- **Setup script ordering**. All VMs start their setup scripts concurrently. If `worker` depends on `control-plane` being ready, the worker's setup must poll until the control plane is available (e.g., `until nc -z control-plane 6443; do sleep 2; done`).
- **Cross-VM env passthrough complexity**. Environment variables in config.yml are static per-VM. For dynamic values (join tokens, generated passwords), use `agent variable set/get` instead.
- **Resource budget**. Multiple VMs consume the sandbox's total resource quota. Three `n1-standard-4` VMs use 12 vCPUs and 45 GB RAM. Check quotas before sizing up.
- **Tab targeting**. Each tab must specify which host it targets via the `hostname` field. A terminal tab for `worker` must set `hostname: worker`.
- **Lifecycle scripts per host**. Setup/check/solve scripts are named with the host suffix: `setup-control-plane`, `check-worker`, etc. Each script runs on its corresponding host.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Kubernetes multi-node cluster setup (kubeadm, k3s server + agent)
- Consul / Nomad / etcd cluster bootstrapping
- Database replication (PostgreSQL primary + replica, MongoDB replica set)
- Ansible automation (control node + managed hosts)
- Network security / firewall rule exercises between hosts

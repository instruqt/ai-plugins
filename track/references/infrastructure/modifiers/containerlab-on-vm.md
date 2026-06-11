# Containerlab on VM

Containerlab creates virtual network topologies by running network operating systems as containers connected by virtual links. Use for networking workshops where learners interact with routers, switches, and network functions.

## What Containerlab Does

Containerlab deploys container-based network nodes (virtual routers, switches, hosts) and wires them together with point-to-point virtual Ethernet links. Each node gets its own network namespace, management IP, and appears as a standalone network device. Learners SSH or console into nodes and configure them like real hardware.

## Requirements

- Docker on the VM (`nested_virtualization: true`).
- `n1-standard-4` minimum. Network vendor images (Arista cEOS, Nokia SR Linux) are resource-intensive. Multi-node topologies with 4+ routers need `n1-standard-8`.
- Network vendor container images must be pre-pulled or available in a registry the VM can reach. Some vendors (Nokia SR Linux, FRRouting) offer free images; others (Arista cEOS, Juniper cRPD) require license-gated downloads.

## Setup Example

```yaml
# config.yml
version: "3"
virtualmachines:
  - name: workstation
    image: instruqt/docker-28-3
    machine_type: n1-standard-4
    shell: /bin/bash
    allow_external_ingress:
      - http
      - https
      - high-ports
    nested_virtualization: true
    provision_ssl_certificate: true
```

```bash
# track_scripts/setup-workstation
#!/bin/bash
set -euxo pipefail

until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done

# Install Containerlab
bash -c "$(curl -sL https://get.containerlab.dev)"

# Pull network OS images
docker pull ghcr.io/nokia/srlinux:latest
docker pull quay.io/frrouting/frr:10.2.1

# Write topology file
mkdir -p /root/lab
cat > /root/lab/topology.yml << 'TOPO'
name: network-lab
topology:
  nodes:
    spine1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
    spine2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
    leaf1:
      kind: linux
      image: quay.io/frrouting/frr:10.2.1
    leaf2:
      kind: linux
      image: quay.io/frrouting/frr:10.2.1
  links:
    - endpoints: ["spine1:e1-1", "leaf1:eth1"]
    - endpoints: ["spine1:e1-2", "leaf2:eth1"]
    - endpoints: ["spine2:e1-1", "leaf1:eth2"]
    - endpoints: ["spine2:e1-2", "leaf2:eth2"]
TOPO

# Deploy the topology
cd /root/lab
containerlab deploy --topo topology.yml

# Wait for all nodes to be running
until [ "$(docker ps --filter label=clab-node-name --format '{{.Status}}' | grep -cv 'Up')" -eq 0 ]; do
  sleep 2
done
```

## Config.yml Implications

| Field | Recommended |
|-------|-------------|
| `machine_type` | `n1-standard-4` for 2-4 nodes, `n1-standard-8` for 5+ nodes |
| `nested_virtualization` | `true` (required -- Containerlab runs Docker containers) |
| `allow_external_ingress` | Include `high-ports` if exposing node management web UIs (e.g., SR Linux JSON-RPC on port 443) |

## Common Pitfalls

- **Network OS images not pre-pulled**. Vendor images are large (1-3 GB). Pulling at deploy time can cause timeouts. Always pull images before `containerlab deploy`.
- **Vendor licensing**. Some network OS images (Arista cEOS, Juniper cRPD) require registration or license files. Verify image availability and license constraints before building a track around a specific vendor platform.
- **Topology deploy timeout**. Complex topologies with many nodes take time to start. The setup script should wait for all containers to reach running state before finishing.
- **Node access method**. Containerlab nodes are accessed via `docker exec` or SSH to management IPs (typically in the `172.20.20.0/24` range). The learner terminal connects to the VM, not directly to network nodes. Provide clear instructions or aliases.
- **Insufficient resources for vendor images**. Nokia SR Linux and Arista cEOS each consume 1-2 GB RAM. A four-router topology can easily need 8+ GB. Under-sized VMs cause containers to restart in a loop.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- BGP peering and route policy configuration
- OSPF area design and troubleshooting
- Network automation with Ansible and NAPALM against virtual routers
- Segment routing / MPLS label switching labs
- Data center leaf-spine fabric build-out
- gNMI/gNOI streaming telemetry collection

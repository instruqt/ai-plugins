# Multi-Container Clustered

Multiple Instruqt containers running instances of the same service to form a cluster -- for example, a three-node database replica set, a Consul server cluster, or an etcd quorum. Each node is a separate container entry with a unique name and a `NODE_ID` environment variable. A workstation container gives the learner CLI access to the cluster.

## Config.yml Example

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/shell
    ports:
      - 8080
    memory: 256
    shell: /bin/bash
    entrypoint: bash
    environment:
      CLUSTER_NODES: "node-1,node-2,node-3"

  - name: node-1
    image: mongo:7
    ports:
      - 27017
    memory: 512
    shell: /bin/bash
    environment:
      NODE_ID: "1"
      CLUSTER_NODES: "node-1,node-2,node-3"

  - name: node-2
    image: mongo:7
    ports:
      - 27017
    memory: 512
    shell: /bin/bash
    environment:
      NODE_ID: "2"
      CLUSTER_NODES: "node-1,node-2,node-3"

  - name: node-3
    image: mongo:7
    ports:
      - 27017
    memory: 512
    shell: /bin/bash
    environment:
      NODE_ID: "3"
      CLUSTER_NODES: "node-1,node-2,node-3"
```

### Variant: three-node Consul cluster

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/shell
    ports:
      - 8500
    memory: 256
    shell: /bin/bash
    entrypoint: bash
    environment:
      CONSUL_HTTP_ADDR: "node-1:8500"

  - name: node-1
    image: hashicorp/consul:1.18
    ports:
      - 8500
      - 8301
      - 8300
    memory: 256
    environment:
      NODE_ID: "1"
      CONSUL_BIND_INTERFACE: eth0

  - name: node-2
    image: hashicorp/consul:1.18
    ports:
      - 8500
      - 8301
      - 8300
    memory: 256
    environment:
      NODE_ID: "2"
      CONSUL_BIND_INTERFACE: eth0

  - name: node-3
    image: hashicorp/consul:1.18
    ports:
      - 8500
      - 8301
      - 8300
    memory: 256
    environment:
      NODE_ID: "3"
      CONSUL_BIND_INTERFACE: eth0
```

## Cluster Formation and Setup Scripts

The core challenge with clustered containers is that all containers start concurrently. No container can assume the others are ready.

### Polling pattern in setup scripts

Each node's setup script must wait for all peers before initiating cluster formation. Run the init command from exactly one node (typically `node-1`).

**setup-node-1** (the initiator):

```bash
#!/bin/bash

# Wait for all cluster members to be reachable
for node in node-1 node-2 node-3; do
  until nc -z "$node" 27017; do
    echo "Waiting for $node..."
    sleep 2
  done
done

# Initiate cluster formation from this node only
mongosh --eval '
  rs.initiate({
    _id: "rs0",
    members: [
      { _id: 0, host: "node-1:27017" },
      { _id: 1, host: "node-2:27017" },
      { _id: 2, host: "node-3:27017" }
    ]
  })
'

# Wait for the replica set to elect a primary
until mongosh --quiet --eval 'rs.status().ok' | grep -q 1; do
  echo "Waiting for replica set to initialize..."
  sleep 2
done
```

**setup-node-2 / setup-node-3** (followers):

```bash
#!/bin/bash

# Wait for the cluster to be formed by node-1
until mongosh --host node-1 --quiet --eval 'rs.status().ok' | grep -q 1; do
  echo "Waiting for cluster initialization..."
  sleep 2
done
```

**setup-workstation**:

```bash
#!/bin/bash

# Wait for the cluster to be healthy before the learner starts
until mongosh --host node-1 --quiet --eval 'rs.status().ok' | grep -q 1; do
  echo "Waiting for cluster..."
  sleep 2
done

echo "Cluster is ready."
```

### Health-gating pattern

The workstation's setup script should gate on the cluster being fully operational. This prevents the learner from seeing a partially formed cluster:

```bash
#!/bin/bash

# Wait for quorum -- at least 2 of 3 nodes report healthy
READY=0
until [ "$READY" -ge 2 ]; do
  READY=0
  for node in node-1 node-2 node-3; do
    if nc -z "$node" 27017 2>/dev/null; then
      READY=$((READY + 1))
    fi
  done
  [ "$READY" -lt 2 ] && sleep 2
done
```

## When to Use

- Track teaches cluster management, quorum concepts, replication, or consensus algorithms.
- Learners need to interact with individual cluster members (inspecting per-node state, triggering failover, adding/removing members).
- The clustered service runs well in containers without Docker or systemd dependencies.
- Track teaches operational procedures: rolling upgrades, node replacement, backup/restore from a replica.

## When NOT to Use

- The clustered service needs Docker, systemd, or kernel features -- use `multi-vm.md` instead.
- Three or more nodes at 512 MB each plus a workstation exceeds the sandbox resource budget -- use VMs for memory-heavy services.
- The cluster is just background infrastructure the learner never inspects directly -- a single container with the service is simpler.
- The service's container image does not support running as a non-init process or requires privileged mode.

## Common Pitfalls

- **Race conditions during cluster init**. All containers start at the same time. If the init script runs before peers are listening, it will fail. Always poll with `nc -z` or equivalent before attempting cluster formation.
- **Multiple init attempts**. If more than one node tries to initiate the cluster, it can corrupt state. Designate exactly one node (typically `node-1`) as the initiator. Other nodes' setup scripts should only wait.
- **Leader election timing**. After `rs.initiate()` or equivalent, the cluster needs time to elect a leader. Add a second poll loop that waits for the leader election to complete before declaring the cluster ready.
- **Learner sees partial state**. If the workstation's setup script finishes before the cluster is fully formed, the learner's first command may fail. Gate the workstation setup on cluster health.
- **Memory pressure**. Three identical service containers can consume significant memory. Size conservatively -- a MongoDB node in a tutorial context often runs fine at 384-512 MB, but three at 512 MB is 1.5 GB before the workstation.
- **Port collisions are not a problem**. Each container has its own network namespace. All three nodes can listen on port 27017 without conflict -- they are reached as `node-1:27017`, `node-2:27017`, `node-3:27017`.
- **Environment variables for discovery**. Pass `CLUSTER_NODES` as a comma-separated list to each container so setup scripts can iterate over peers without hardcoding.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- MongoDB replica set administration (initiate, failover, add/remove members)
- Consul or etcd cluster bootstrapping and service discovery
- Redis Sentinel or Redis Cluster setup and failover
- Elasticsearch cluster management (shard allocation, rolling restarts)
- Distributed consensus concepts (Raft visualization, split-brain scenarios)

# Cross-VM Networking

Evaluates whether multi-VM tracks correctly handle inter-host communication, hostname discovery, state sharing, and file transfer between sandbox hosts.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Multi-VM track with no coordination -- VMs cannot find each other, no shared state mechanism, scripts assume hardcoded IPs |
| 2 | Below Standard | VMs can reach each other by hostname but state sharing is ad-hoc -- scripts write files to /tmp and hope the other VM reads them |
| 3 | Adequate | Hostnames shared via environment: block and basic state shared via agent variables, but complex data passed as flattened strings |
| 4 | Good | VMs can find each other via environment variables; state shared using agent variables for simple values; file transfer via SCP where needed (production baseline) |
| 5 | Excellent | Coordination pattern chosen appropriately for the data complexity -- agent variables for simple flags, JSON-broker sidecar for structured data, SCP for files; patterns documented in config comments |

## Guidance

When a track has multiple VMs or containers, they need to discover each other and share state. There are several patterns, each suited to different complexity levels.

### Hostname Discovery via environment: Block

The simplest coordination: each VM gets the hostnames of other VMs as environment variables.

Good -- mutual hostname awareness:

```yaml
virtualmachines:
- name: server
  machine_type: n1-standard-2
  image: ubuntu-22-04
  environment:
    CLIENT_HOST: client
- name: client
  machine_type: n1-standard-1
  image: ubuntu-22-04
  environment:
    SERVER_HOST: server
```

VMs in the same sandbox can resolve each other by name (the Instruqt network handles DNS), but explicit environment variables make scripts clearer and avoid hardcoding hostnames.

### Agent Variables for Simple State

The Instruqt agent on each VM can store and retrieve key-value pairs that persist across challenges. Use this for simple values like IP addresses, generated passwords, or status flags.

Good -- sharing a generated token between VMs:

```bash
# On server (setup.sh)
TOKEN=$(openssl rand -hex 16)
agent variable set TOKEN "$TOKEN"

# On client (setup.sh)
TOKEN=$(agent variable get TOKEN)
echo "Server token: $TOKEN"
```

### JSON-Broker Sidecar for Structured Data

When VMs need to share structured data (lists, nested objects, configuration blocks), a JSON file served via a simple HTTP endpoint is more reliable than flattening everything into agent variables.

Good -- JSON broker pattern:

```bash
# On server (setup.sh) -- write structured state
cat > /tmp/cluster-info.json <<'JSONEOF'
{
  "cluster_name": "demo-cluster",
  "api_endpoint": "https://server:6443",
  "nodes": ["server", "worker-1", "worker-2"],
  "token": "abc123"
}
JSONEOF
# Serve it (background process or simple HTTP server)
cd /tmp && python3 -m http.server 9999 &

# On client (setup.sh) -- consume structured state
CLUSTER_INFO=$(curl -s http://server:9999/cluster-info.json)
API_ENDPOINT=$(echo "$CLUSTER_INFO" | jq -r '.api_endpoint')
```

### SCP for File Transfer

When VMs need to share files (certificates, kubeconfigs, binaries), SCP is the most straightforward approach. Ensure SSH keys are configured.

Good -- transferring a certificate:

```bash
# On server (setup.sh) -- generate cert and make it available
openssl req -x509 -newkey rsa:2048 -keyout /tmp/tls.key -out /tmp/tls.crt -days 1 -nodes -subj "/CN=server"

# On client (setup.sh) -- copy the cert
scp -o StrictHostKeyChecking=no server:/tmp/tls.crt /usr/local/share/ca-certificates/server.crt
update-ca-certificates
```

### Choosing the Right Pattern

| Data Type | Pattern | Example |
|-----------|---------|---------|
| Simple string/flag | Agent variable | Generated password, cluster join token |
| Hostname/IP | environment: block | Server address for client to connect to |
| Structured config | JSON broker | Cluster metadata with multiple fields |
| Binary/large file | SCP | TLS certificates, kubeconfig, binaries |

Bad -- hardcoded IPs:

```bash
# IPs are dynamic in Instruqt sandboxes
curl http://10.0.0.2:8080/api
```

Bad -- complex data crammed into agent variables:

```bash
# Agent variables are for simple values, not JSON blobs
agent variable set CLUSTER_CONFIG '{"name":"demo","nodes":["a","b","c"],"endpoints":{"api":"https://...","dashboard":"https://..."}}'
```

Bad -- no coordination in a multi-VM track:

```yaml
virtualmachines:
- name: server
  machine_type: n1-standard-2
  image: ubuntu-22-04
- name: client
  machine_type: n1-standard-1
  image: ubuntu-22-04
  # No environment: block, no way for client to know about server
```

## What to Watch For

- Multi-VM tracks where scripts use hardcoded IPs -- sandbox IPs are dynamic
- Agent variables used for complex structured data that would be cleaner as JSON
- SCP commands without `StrictHostKeyChecking=no` -- the hosts have not seen each other before
- Missing environment: blocks that leave VMs unaware of each other's hostnames
- Race conditions where one VM's setup script assumes another VM is already fully provisioned -- use wait loops or agent variable flags
- Setup scripts on VM B that depend on VM A's setup completing first -- Instruqt runs setup scripts in parallel across VMs

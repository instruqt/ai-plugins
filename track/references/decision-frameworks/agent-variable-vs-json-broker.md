# Agent Variable vs JSON Broker vs Cross-VM Env Passthrough

Choosing the right mechanism for sharing state between sandbox hosts in multi-VM or complex tracks.

## Decision Criteria

### Agent variable (`agent variable set/get`)

Use for single string values, especially when the value needs to be interpolated into assignment text via `[[ Instruqt-Var ]]` syntax.

- Set from any lifecycle script: `agent variable set MY_TOKEN abc123`
- Read from any lifecycle script: `agent variable get MY_TOKEN`
- Interpolated in assignments: `[[ Instruqt-Var key="MY_TOKEN" hostname="workstation" ]]`
- Best for: generated passwords, cluster join tokens, dynamic URLs, values displayed to learners.

### JSON broker (serialized payload via agent variable or shared file)

Use for structured data that multiple VMs or scripts need to consume.

- Store as serialized JSON in an agent variable or a file on a shared path.
- Downstream consumers parse with `jq` or equivalent.
- Best for: multi-field configuration (e.g., database credentials + host + port), cluster membership lists, complex state that does not fit a single string.

Example:

```bash
# On the producer VM
agent variable set CLUSTER_CONFIG "$(jq -n \
  --arg token "$JOIN_TOKEN" \
  --arg endpoint "$API_ENDPOINT" \
  --arg ca_hash "$CA_HASH" \
  '{token: $token, endpoint: $endpoint, ca_hash: $ca_hash}')"

# On the consumer VM
CLUSTER_CONFIG=$(agent variable get CLUSTER_CONFIG)
JOIN_TOKEN=$(echo "$CLUSTER_CONFIG" | jq -r '.token')
API_ENDPOINT=$(echo "$CLUSTER_CONFIG" | jq -r '.endpoint')
```

### Cross-VM environment passthrough (config.yml `environment:` block)

Use for static configuration known at track design time: hostnames, roles, fixed ports, feature flags.

- Defined in config.yml, available as shell environment variables on the target VM.
- Cannot carry dynamic or runtime-generated values.
- Best for: peer hostnames, node roles (server/agent), cluster topology constants.

Example:

```yaml
virtualmachines:
  - name: server
    environment:
      ROLE: server
      WORKER_HOSTNAME: worker
  - name: worker
    environment:
      ROLE: agent
      SERVER_HOSTNAME: server
```

## Trade-offs

| Factor | Agent Variable | JSON Broker | Cross-VM Env |
|--------|---------------|-------------|-------------|
| Value type | Single string | Structured (JSON) | Single string |
| Dynamic values | Yes | Yes | No (static only) |
| Assignment interpolation | Yes (`[[ Instruqt-Var ]]`) | No | No |
| Setup complexity | Minimal | Moderate (jq parsing) | Minimal (config.yml) |
| Timing dependency | Producer must run before consumer reads | Same as agent variable | Available at VM boot |
| Multi-consumer | Yes (any host can read) | Yes (any host can read) | Per-VM (each VM gets its own block) |
| Debugging | `agent variable get` from terminal | Same + jq parsing | `env | grep` on the VM |

## Recommendation

These mechanisms are **not mutually exclusive**. Complex tracks commonly use all three simultaneously:

1. **Cross-VM env** for static topology (hostnames, roles) -- set once in config.yml, never changes.
2. **Agent variable** for dynamic single values (join tokens, generated passwords) -- especially when values appear in assignment text.
3. **JSON broker** for structured payloads consumed by multiple downstream scripts -- when a single agent variable would require awkward string packing.

**Default to agent variables** for most dynamic state sharing. Reach for JSON broker only when the payload is genuinely structured. Use cross-VM env for anything known at design time.

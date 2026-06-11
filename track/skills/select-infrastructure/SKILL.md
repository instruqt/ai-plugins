# Select Infrastructure

Recommend the right sandbox infrastructure for a track based on its requirements and priorities.

## Progress reporting

Do not create your own top-level task list. The invoking command owns the user-facing task list. If this skill represents a single user-visible step in that list, your work maps to one entry. If you need finer-grained progress, update the existing task list rather than starting a new one.

## Paths

- Infrastructure index: `${CLAUDE_PLUGIN_ROOT}/references/infrastructure/_index.md`
- Decision frameworks: `${CLAUDE_PLUGIN_ROOT}/references/decision-frameworks/_index.md`

## Inputs

The invoking agent provides:

1. **What the track teaches** — the product, technology, or concepts being covered.
2. **What software must run** — databases, web servers, Docker, Kubernetes, cloud CLIs, GUI applications, etc.
3. **What cloud resources are needed** — AWS, Azure, GCP accounts, or none.
4. **Priorities** — the relative importance of these factors (not all tracks weight them equally):
   - **Startup speed** — how fast the sandbox must be ready (affects container vs VM, custom images vs setup scripts).
   - **Realism** — how close to a production environment (affects VM vs container, managed K8s vs Kind/k3s).
   - **Author complexity** — how much setup/maintenance the track author takes on (affects multi-VM vs single-VM, custom images vs heredoc).
   - **Cost** — resource consumption per session (affects VM size, number of hosts, cloud accounts).
   - **Learner UX** — the quality of the interactive experience (affects tab selection, GUI method, IDE choice).

## Workflow

### Step 1: Determine Base Compute

Start with the fundamental constraint: **does the track need Docker socket access?**

Read `decision-frameworks/container-vs-vm.md` and apply:

```
Needs Docker, Compose, Kind, k3s, or Containerlab?  --> VM
Needs systemd?                                       --> VM
Needs kernel features (iptables, eBPF, cgroups)?     --> VM
Needs GUI desktop?                                   --> VM
Needs >1 GB memory?                                  --> VM (usually)
CLI-only with cloud tools?                           --> Container
CLI-only with basic tools?                           --> Container
Heavy services + light CLI?                          --> Container + VM hybrid
```

Then narrow within the category. Read the relevant base-compute files from `infrastructure/base-compute/`:

- **Container path:** single-container vs multi-container (multi-tier, clustered, sidecar)
- **VM path:** bare VM vs VM+Docker vs VM+Compose vs VM+Kind/k3s vs fat-stack
- **Multi-host:** multi-VM vs container+VM hybrid
- **None:** sandbox-preset (if a preset matches perfectly)

### Step 2: Determine Cloud Accounts

If the track needs real cloud resources:

1. Read `decision-frameworks/cloud-provider-selection.md`
2. Identify which provider(s) are needed
3. Read the matching pattern from `infrastructure/cloud-accounts/`
4. If managed Kubernetes is needed, read from `infrastructure/managed-kubernetes/`
5. If runtime provisioning is needed, read from `infrastructure/runtime-provisioning/`

### Step 3: Select Modifiers

Based on what the track needs on top of the base compute:

1. Read `infrastructure/modifiers/_index.md`
2. Select applicable modifiers:
   - **VM-only:** Docker, Compose, Kind/k3s, Containerlab
   - **General:** custom images (Packer or Docker), SSL certificates, private repo access, account pool checkout
3. For image decisions, check `decision-frameworks/custom-image-vs-heredoc.md`
4. For VM sizing, check `decision-frameworks/machine-type-vs-memory-cpus.md`

### Step 4: Select Learner UX

Determine what tabs and interfaces the learner needs:

1. Read `decision-frameworks/tab-type-selection.md`
2. Every track needs at least a terminal tab
3. If the track has a web UI: service tab (most common) or virtual browser (if iframe won't work)
4. If the track needs file editing: read `decision-frameworks/editor-selection.md` — native editor tab vs code-server
5. If the track needs GUI desktop: read `decision-frameworks/gui-access-method.md` — Guacamole RDP vs VNC vs KasmVNC vs noVNC
6. Read the specific UX files from `infrastructure/learner-ux/` for setup details

### Step 5: Produce Recommendation

Output a structured recommendation:

```
## Infrastructure Recommendation

### Base Compute
- Pattern: [pattern name]
- Rationale: [why this pattern fits]

### Cloud Accounts
- Provider(s): [AWS/Azure/GCP/none]
- Pattern: [pattern name]
- Rationale: [why]

### Modifiers
- [modifier 1]: [why needed]
- [modifier 2]: [why needed]

### Learner UX
- Tabs: [list of tabs with types]
- Rationale: [why these tabs]

### Config.yml Skeleton
[minimal config.yml showing the recommended structure]

### Trade-offs and Alternatives
- [what was considered and rejected, and why]
- [what priority shifts would change the recommendation]
```

## Priority-Driven Adjustments

When priorities conflict, use these guidelines:

| Priority | Favors | At the cost of |
|----------|--------|----------------|
| Startup speed | Containers, custom images, presets | Realism, flexibility |
| Realism | VMs, managed K8s, multi-VM | Startup speed, cost |
| Low author complexity | Single host, presets, containers | Realism, learner UX |
| Low cost | Containers, smaller VMs, fewer hosts | Realism, learner UX |
| Best learner UX | Code-server, service tabs, virtual browsers | Author complexity, cost |

## Anti-Pattern Awareness

Before finalizing, check that the recommendation does not fall into known anti-patterns:

- Read `discover-best-practices` for anti-patterns relevant to the chosen infrastructure
- Common traps: secrets in environment blocks, missing cloud resource blocks, sleep instead of polling, baked credentials

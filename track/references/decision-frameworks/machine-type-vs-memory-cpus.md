# Machine Type vs Memory/CPUs

Deciding whether to size VMs using `machine_type` or the `memory` + `cpus` fields in config.yml.

## Decision Criteria

### Use `machine_type` (default choice):

- Canonical sizing knob across all major Instruqt organizations.
- Maps directly to GCP Compute Engine shapes with predictable performance characteristics.
- Preferred by the Instruqt platform -- the documentation, examples, and tooling all default to it.
- Easier to reason about: `n1-standard-4` is universally understood as 4 vCPU / 15 GB.

### Use `memory` + `cpus` only when:

- The track needs a non-standard CPU-to-memory ratio not covered by any machine type family.
- Matching a legacy track configuration that already uses `memory` + `cpus`.
- The specific resource amount matters more than the GCP shape (rare).

## Machine Type Catalog

See `track-format/machine-types.md` for the full catalog of supported GCP machine types (E2, F1/G1, N1, N2, N2D, and others) with vCPU and memory specs — it is the single source of truth for the list. This document covers only the decision of *whether* to use `machine_type` vs `memory`/`cpus`.

## Trade-offs

| Factor | `machine_type` | `memory` + `cpus` |
|--------|---------------|-------------------|
| Clarity | Self-documenting (e.g., `n1-standard-4`) | Requires mental math |
| Platform alignment | Preferred by Instruqt docs and tooling | Supported but secondary |
| Flexibility | Constrained to GCP shapes | Arbitrary ratios |
| Cross-org consistency | Standard vocabulary | Numbers vary in meaning |
| Performance predictability | GCP guarantees per shape | Platform maps to nearest shape |

## Recommendation

**Default to `machine_type`.** It is the canonical, self-documenting, platform-preferred sizing knob. Use `n1-standard-2` as the starting point for most tracks, and size up based on workload testing.

Fall back to `memory` + `cpus` only when matching a legacy configuration or when no machine type family provides the specific ratio needed.

### Quick sizing guide

| Workload | Recommended machine type |
|----------|------------------------|
| CLI exercises, simple scripting | `n1-standard-1` or `e2-small` |
| General workshops (Docker, single app) | `n1-standard-2` |
| Kubernetes (k3s/Kind), Compose stacks | `n1-standard-4` |
| JVM/database-heavy, multi-service | `n2-highmem-4` or `n1-standard-8` |
| Large clusters, data processing | `n1-standard-8` or larger |
| Lightweight auxiliary VMs | `e2-small` or `e2-medium` |

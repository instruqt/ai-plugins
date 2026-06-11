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

### E2 Series (cost-efficient)

| Name | vCPUs | Memory (GB) |
|------|------:|------------:|
| e2-micro | 0.25 | 1 |
| e2-small | 0.5 | 2 |
| e2-medium | 1 | 4 |
| e2-standard-2 | 2 | 8 |
| e2-standard-4 | 4 | 16 |
| e2-standard-8 | 8 | 32 |
| e2-standard-16 | 16 | 64 |
| e2-standard-32 | 32 | 128 |
| e2-highmem-2 | 2 | 16 |
| e2-highmem-4 | 4 | 32 |
| e2-highmem-8 | 8 | 64 |
| e2-highmem-16 | 16 | 128 |
| e2-highcpu-2 | 2 | 2 |
| e2-highcpu-4 | 4 | 4 |
| e2-highcpu-8 | 8 | 8 |
| e2-highcpu-16 | 16 | 16 |
| e2-highcpu-32 | 32 | 32 |

### F1/G1 Series (shared-core, minimal)

| Name | vCPUs | Memory (GB) |
|------|------:|------------:|
| f1-micro | 0.2 | 0.6 |
| g1-small | 0.5 | 1.7 |

### N1 Series (general purpose)

| Name | vCPUs | Memory (GB) |
|------|------:|------------:|
| n1-standard-1 | 1 | 3.75 |
| n1-standard-2 | 2 | 7.50 |
| n1-standard-4 | 4 | 15 |
| n1-standard-8 | 8 | 30 |
| n1-highcpu-2 | 2 | 1.8 |
| n1-highcpu-4 | 4 | 3.6 |
| n1-highcpu-8 | 8 | 7.2 |
| n1-highcpu-16 | 16 | 14.4 |
| n1-highcpu-32 | 32 | 28.8 |
| n1-highmem-2 | 2 | 13 |
| n1-highmem-4 | 4 | 26 |
| n1-highmem-8 | 8 | 52 |
| n1-highmem-16 | 16 | 104 |
| n1-highmem-32 | 32 | 208 |

### N2 Series (balanced performance)

| Name | vCPUs | Memory (GB) |
|------|------:|------------:|
| n2-standard-2 | 2 | 8 |
| n2-standard-4 | 4 | 16 |
| n2-standard-8 | 8 | 32 |
| n2-standard-16 | 16 | 64 |
| n2-standard-32 | 32 | 128 |
| n2-highmem-2 | 2 | 16 |
| n2-highmem-4 | 4 | 32 |
| n2-highmem-8 | 8 | 64 |
| n2-highmem-16 | 16 | 128 |
| n2-highmem-32 | 32 | 256 |
| n2-highcpu-2 | 2 | 2 |
| n2-highcpu-4 | 4 | 4 |
| n2-highcpu-8 | 8 | 8 |
| n2-highcpu-16 | 16 | 16 |
| n2-highcpu-32 | 32 | 32 |

### N2D Series (AMD EPYC)

| Name | vCPUs | Memory (GB) |
|------|------:|------------:|
| n2d-standard-2 | 2 | 8 |
| n2d-standard-4 | 4 | 16 |
| n2d-standard-8 | 8 | 32 |
| n2d-standard-16 | 16 | 64 |
| n2d-standard-32 | 32 | 128 |
| n2d-highmem-2 | 2 | 16 |
| n2d-highmem-4 | 4 | 32 |
| n2d-highmem-8 | 8 | 64 |
| n2d-highmem-16 | 16 | 128 |
| n2d-highmem-32 | 32 | 256 |
| n2d-highcpu-2 | 2 | 2 |
| n2d-highcpu-4 | 4 | 4 |
| n2d-highcpu-8 | 8 | 8 |
| n2d-highcpu-16 | 16 | 16 |
| n2d-highcpu-32 | 32 | 32 |

### Other Supported Types

| Name | vCPUs | Memory (GB) | Notes |
|------|------:|------------:|-------|
| c2-standard-8 | 8 | 32 | Compute-optimized |
| t2a-standard-1 | 1 | 4 | Arm64 (Ampere Altra) |

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

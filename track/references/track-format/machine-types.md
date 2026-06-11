# Machine Types

GCP Compute Engine machine types used in `machine_type` fields for virtual machine sandbox hosts. Using `machine_type` is preferred over specifying `memory` and `cpus` separately.

## Structure

Machine type is set in the sandbox host configuration:

```yaml
virtualmachines:
  - name: workstation
    image: instruqt-alma-9-base
    machine_type: n1-standard-2
```

## Fields

The `machine_type` field accepts a GCP machine type string. The platform supports the following types:

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

## Examples

### Standard Workstation

Most tracks should start with `n1-standard-2`:

```yaml
virtualmachines:
  - name: workstation
    image: instruqt-alma-9-base
    machine_type: n1-standard-2
```

### High-Memory Workload

For workloads needing significant memory (databases, JVM apps):

```yaml
virtualmachines:
  - name: db-server
    image: instruqt-ubuntu-22-04-base
    machine_type: n2-highmem-8
```

### Minimal Footprint

For lightweight auxiliary VMs:

```yaml
virtualmachines:
  - name: jumpbox
    image: instruqt-alma-9-base
    machine_type: e2-small
```

### Arm64

For ARM-specific content:

```yaml
virtualmachines:
  - name: arm-host
    image: instruqt-ubuntu-22-04-base
    machine_type: t2a-standard-1
```

## Notes

- `machine_type` is preferred over specifying `memory` and `cpus` separately.
- `n1-standard-2` is a good default for most use cases.
- Instruqt charges based on resource usage -- choose the smallest type that meets your workload needs.
- Use `free -m` on Linux hosts to check current memory usage during testing.
- The maximum supported configuration is 32 vCPUs / 256 GB memory (n2-highmem-32 or n2d-highmem-32).

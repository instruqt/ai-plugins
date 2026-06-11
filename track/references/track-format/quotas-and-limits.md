# Quotas and Limits

Platform-level quotas and resource limits that apply to Instruqt sandbox environments. These constrain what sandbox hosts, cloud projects, and scripts can use.

## Structure

Limits are enforced by the platform and cannot be overridden in track configuration. They apply per-sandbox (per play session).

## Fields

### Sandbox Hosts

| Resource | Limit |
|----------|-------|
| Virtual machines | Up to 32 vCPUs and 208 GB memory (depends on machine type) |
| Container memory | Up to 8 GB (8192 MB) per container |
| Total containers + website services | 15 combined per sandbox |

### GCP Project Limits

| Resource | Limit |
|----------|-------|
| CPU quota | 16 vCPUs per play (does not apply to sandbox hosts themselves) |
| Blocked APIs | `serviceusage.googleapis.com` (prevents learner from enabling additional services) |
| Blocked machine types | Memory-optimized (M1, M2), Compute-optimized (C2), Accelerator-optimized (A2) |
| GPU quota | Disabled (contact account manager for exceptions) |

### Azure Subscription Limits

| Resource | Limit |
|----------|-------|
| VM types | General-Purpose only, 16 vCPU or less |
| GPU/accelerated compute | Not allowed |

Supported Azure VM families:

- basicAFamily
- standardA0_A7Family
- standardAv2Family
- standardBSFamily
- standardBsv2Family
- standardDASv4Family, standardDASv5Family, standardDASv6Family
- standardDAv4Family
- standardDCSv2Family
- standardDDSv4Family, standardDDv4Family
- standardDFamily
- standardDLDSv5Family
- StandardDpsv6Family
- standardDSFamily, standardDSv2Family, standardDSv3Family, standardDSv4Family
- standardDv2Family, standardDv3Family, standardDv4Family

### AWS Account Limits

| Resource | Limit |
|----------|-------|
| EC2 instances | General-Purpose only |
| GPU/accelerated compute | Not allowed |
| Route53 | Domain name registration not allowed |

Supported AWS EC2 instance types (wildcard notation):

- `t*.nano` through `t*.*large`
- `m*.nano` through `m*.*large`
- `a1.nano` through `a1.*large`
- `i*.*large`

## Examples

### Staying Within Container Limits

A sandbox with 3 containers and 2 website services uses 5 of the 15 combined slots:

```yaml
containers:
  - name: app
    memory: 4096  # 4 GB -- within the 8192 MB max
  - name: database
    memory: 2048
  - name: cache
    memory: 512
```

### GCP Project Awareness

When a track provisions GCP resources for the learner, the 16 vCPU quota applies to resources created inside the GCP project (e.g., Compute Engine VMs the learner launches), not to the sandbox host VMs defined in the track config.

## Notes

- Container memory is specified in MB in track configuration.
- The 15-slot limit counts containers and website services together -- VMs are separate.
- Cloud provider limits apply to resources the learner creates within provisioned accounts, not to the sandbox host infrastructure.

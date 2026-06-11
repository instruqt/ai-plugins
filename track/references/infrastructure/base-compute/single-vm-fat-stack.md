# Single VM Fat Stack

One large virtual machine running a self-contained multi-service deployment. High memory and CPU (typically `n1-standard-8` or larger, sometimes up to `n1-highmem-16` with 104 GB RAM) to accommodate an entire product stack on a single box -- multiple Docker containers, databases, application servers, monitoring, all co-located.

See `modifiers/docker-compose-on-vm.md` for Compose orchestration details and `modifiers/custom-image-packer.md` for building pre-baked images (almost always required for this pattern).

## Config.yml Example

```yaml
version: "3"
virtualmachines:
  - name: platform
    image: projects/my-instruqt-project/global/images/my-product-stack-v3
    machine_type: n1-standard-8
    shell: /bin/bash
    nested_virtualization: true
    allow_external_ingress:
      - http
      - https
      - high-ports
    provision_ssl_certificate: true
```

### Why a custom image

A fat stack from a bare OS image would spend 5-15 minutes in setup pulling images and installing dependencies. Custom Packer images pre-bake everything so the VM boots with services ready to start (or already running via systemd).

### Typical setup script (with custom image)

When using a pre-baked image, the setup script is lightweight -- just start services and wait for readiness:

```bash
cd /opt/platform && docker compose up -d

# Wait for the full stack to be healthy
until curl -sf http://localhost:8080/health > /dev/null; do
  sleep 3
done
```

## When to Use

- The product requires many co-located services that cannot easily be split across hosts (enterprise software platforms, all-in-one deployment models).
- Track needs a realistic production-like environment on a single box for the learner to explore, configure, or troubleshoot.
- The vendor's recommended deployment model is a single-host install (common with enterprise security, observability, and data platforms).
- Simpler author experience than multi-VM -- one setup script, one host, no cross-VM coordination.

## When NOT to Use

- Only 2-3 lightweight containers -- use `single-vm-docker-compose.md` with a standard machine type instead.
- Services are naturally distributed across hosts and the track teaches that distribution -- use `multi-vm.md`.
- The stack fits in 8 GB RAM or less -- you probably do not need a fat VM. Start with `n1-standard-4` and only scale up if needed.
- Cost is a concern for high-volume tracks -- large VMs are expensive per-sandbox. Explore whether a lighter pattern can achieve the same learning objectives.

## Common Pitfalls

- **Not using a custom Packer image**. This is the most common mistake with fat stacks. Pulling 10+ Docker images and installing packages at sandbox start is unreliable and slow. Always build a custom image for production fat-stack tracks (see `modifiers/custom-image-packer.md`).
- **Startup timeouts**. Even with a custom image, starting many services takes time. Use robust health-check loops in setup scripts. Instruqt has a 10-minute setup timeout -- complex stacks can hit this if services are slow to initialize.
- **Memory pressure and OOM kills**. A stack that needs 12 GB at steady state should run on a VM with at least 16 GB to allow headroom for setup, image loading, and learner activity. Monitor `dmesg` for OOM events during testing.
- **Disk space**. Large stacks with many Docker images can exhaust the default disk. Use `disk_size` in config.yml if the image cache exceeds 20 GB.
- **Port sprawl**. With many services, port assignments become hard to track. Document all port mappings in a comment block at the top of the compose file and keep a consistent scheme (e.g., 8080 for web UI, 3000 for API, 5432 for database, 9090 for monitoring).
- **Debugging difficulty**. When something fails in a 10-service stack, diagnosis is harder. Include a health-check summary endpoint or script that reports the status of each component, and use it in check scripts.
- **Image versioning**. Fat-stack custom images are expensive to rebuild. Pin all component versions explicitly and tag Packer images with a version number so you can roll back.

## Machine Type Reference

| Machine type | vCPU | RAM | Typical use case |
|-------------|------|-----|-----------------|
| `n1-standard-8` | 8 | 30 GB | Medium stacks (5-8 services, no JVM-heavy workloads) |
| `n1-highmem-8` | 8 | 52 GB | Memory-heavy stacks (multiple JVM services, large databases) |
| `n1-standard-16` | 16 | 60 GB | Large stacks (10+ services, parallel processing) |
| `n1-highmem-16` | 16 | 104 GB | Enterprise platforms with heavy memory requirements |

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Enterprise platform administration (all-in-one platform install with web UI, API, database, workers, monitoring)
- Security operations center simulation (SIEM + log collectors + threat intel + case management)
- Observability stack workshops (application + Prometheus + Grafana + Loki + Tempo + OpenTelemetry Collector)
- Data pipeline workshops (ingestion + processing + storage + visualization all on one host)
- Troubleshooting exercises (pre-broken multi-service stack that the learner must diagnose and fix)

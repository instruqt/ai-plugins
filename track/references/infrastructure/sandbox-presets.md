# Sandbox Preset

A pre-built environment selected in the Instruqt UI. No `config.yml` and no `track_scripts/` directory needed -- the sandbox comes ready to use with pre-installed tooling and running services.

## Directory Structure

There is no `config.yml`. The track repository contains only challenge directories with assignment files and optional per-challenge lifecycle scripts.

```
my-track/
  track.yml
  01-first-challenge/
    assignment.md
    check-<hostname>
    solve-<hostname>
  02-second-challenge/
    assignment.md
    check-<hostname>
    solve-<hostname>
```

The hostname used in lifecycle script names comes from the preset definition (e.g., `kubernetes-vm`, `server`, `console`).

## When to Use

- The preset's pre-installed tools and services match exactly what the track needs.
- The track is a click-through demo against a managed UI (e.g., Kibana, monitoring dashboards).
- No custom images, multi-VM topologies, secrets injection, or per-track sizing are needed.
- The organization provides and maintains the preset.

## When NOT to Use

- Track needs any setup scripts (`track_scripts/`).
- Track needs custom images, multiple VMs, cloud accounts, or secrets injection.
- Track needs specific VM sizing different from what the preset provides.
- You need fine-grained control over the environment.

Start with `config.yml` if there is any doubt. See `decision-frameworks/preset-vs-custom-config.md` for the full decision framework.

## Limitations

| Capability | Preset | Custom config.yml |
|-----------|--------|-------------------|
| Custom Docker images | No | Yes |
| Custom VM images (Packer) | No | Yes |
| Multiple VMs | Only if preset includes them | Yes |
| Cloud accounts (AWS/GCP/Azure) | Only if preset includes them | Yes |
| Secrets injection | No | Yes |
| Per-track machine sizing | No | Yes |
| Setup scripts (`track_scripts/`) | No | Yes |
| Per-challenge lifecycle scripts | Yes | Yes |
| Custom networking / ports | No | Yes |

## Preset Families

Presets are organized by product or technology. Common families include:

- **Elastic/ECK** — Elasticsearch, Kibana, APM pre-deployed on Kubernetes.
- **Managed VM** — VMs with specific product stacks pre-installed (high-memory variants available).
- **Cloud-native** — VMs with cloud CLIs, kubectl, helm pre-installed.
- **Kubernetes** — k3s or Kind clusters pre-configured and running.

Check the Instruqt UI for the current preset catalog available to your organization.

## Common Pitfalls

- **Preset drift**. Presets are maintained by the org that owns them. Version updates can change installed tools, hostnames, or service configurations without notice.
- **No setup scripts**. Without `track_scripts/`, there is no hook to customize the environment at track start. All customization must happen in per-challenge scripts.
- **Hostname discovery**. The preset determines hostnames. Check the preset documentation or test the sandbox to find the correct hostname for lifecycle scripts and tabs.
- **Locked sizing**. If the preset provisions an `n1-standard-2` VM but the track needs more resources, there is no way to override it short of switching to a custom config.yml.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Monitoring and observability demos against pre-deployed Kibana/Grafana
- Kubernetes policy workshops on pre-configured clusters
- Click-through product tours against managed UIs
- Cloud-native tool introductions using pre-installed CLI environments

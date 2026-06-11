# Preset vs Custom Config

Deciding whether to use a sandbox preset or write a custom `config.yml` for track infrastructure.

## Decision Criteria

### Use a sandbox preset when ALL of these are true:

- The preset's pre-installed tools and services match exactly what the track needs.
- The track is a click-through demo against a managed UI (e.g., Kibana, Temporal UI).
- No custom setup scripts are needed at track start (`track_scripts/`).
- No custom images, secrets, or per-track sizing are required.
- The preset is maintained by your organization or a trusted partner.

### Use a custom config.yml when ANY of these are true:

- The track needs setup scripts to configure the environment.
- A custom Docker or VM image is required.
- Multiple VMs or specific VM topologies are needed.
- Cloud accounts (AWS, GCP, Azure) must be provisioned.
- Secrets or environment variables need injection.
- Specific machine sizing beyond the preset's default is required.
- Custom networking, ports, or ingress rules are needed.

## Trade-offs

| Factor | Sandbox Preset | Custom config.yml |
|--------|---------------|-------------------|
| Setup time | Fastest -- environment is pre-built | Depends on setup script complexity |
| Flexibility | Locked to preset configuration | Full control over all infrastructure |
| Custom images | Not supported | Supported |
| Multiple VMs | Only if preset includes them | Yes |
| Cloud accounts | Only if preset includes them | Yes |
| Secrets injection | Not supported | Supported |
| Machine sizing | Fixed by preset | Configurable per-track |
| Setup scripts | No `track_scripts/` | Full lifecycle script support |
| Maintenance | Preset owner handles updates | Track author owns infrastructure |
| Version stability | Preset updates may change behavior | Pinned to your image versions |

## Recommendation

**Start with `config.yml` if there is any doubt.** Converting from a preset to a custom config later means building the entire infrastructure definition from scratch. Converting from a custom config to a preset (if one becomes available) is simply deleting `config.yml` and `track_scripts/`.

Presets are the right choice only for pure click-through demos where the preset's tooling matches exactly and no customization is needed. In practice, most tracks benefit from the control that a custom `config.yml` provides.

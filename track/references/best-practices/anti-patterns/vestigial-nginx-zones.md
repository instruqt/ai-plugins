---
severity: advisory
---
# Vestigial Nginx-Zones Container

Copying the nginx-zones container from Consul HVD (HashiCorp Validated Designs) configurations into new tracks.

## Why It's Bad

The nginx-zones container is left over from an earlier Consul architecture. It serves no function in current track designs. Including it inflates sandbox resource allocation (CPU, memory, network) for zero benefit and adds a container that must be pulled, started, and managed during setup.

## What to Do Instead

Omit the nginx-zones container entirely when building new tracks. If porting from an existing Consul HVD config, audit every container and remove those that are not actively used by the lab.

## Examples

Bad -- carrying over nginx-zones from a template:

```yaml
# docker-compose.yml
services:
  consul:
    image: hashicorp/consul:1.17
    # ...
  nginx-zones:
    image: nginx:alpine
    volumes:
      - ./zones.conf:/etc/nginx/conf.d/default.conf
    # Vestigial -- serves no function
```

Good -- only include containers the lab actually uses:

```yaml
# docker-compose.yml
services:
  consul:
    image: hashicorp/consul:1.17
    # ...
```

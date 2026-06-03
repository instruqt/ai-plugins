# Docker Compose on VM

Docker Compose orchestrating a multi-service stack on a single virtual machine. All containers share one VM and communicate over a Compose network. Use when the track needs multiple cooperating services (web + database + cache, microservices) without the overhead of separate Instruqt hosts per container.

## Requirements

- Everything from the Docker on VM modifier: `nested_virtualization: true`, `allow_external_ingress`, etc.
- Docker Compose v2 (the `docker compose` plugin) ships with Docker CE 20.10+ and all `instruqt/docker-*` images. No separate install needed.
- `n1-standard-4` or larger -- multiple containers sharing one VM need headroom.

## Setup Example

```yaml
# config.yml
version: "3"
virtualmachines:
  - name: workstation
    image: instruqt/docker-28-3
    machine_type: n1-standard-4
    shell: /bin/bash
    allow_external_ingress:
      - http
      - https
      - high-ports
    nested_virtualization: true
    provision_ssl_certificate: true
```

```bash
# track_scripts/setup-workstation
#!/bin/bash
set -euxo pipefail

until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done

# Write the Compose file
mkdir -p /root/app
cat > /root/app/compose.yml << 'COMPOSE'
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    depends_on:
      api:
        condition: service_healthy

  api:
    image: python:3.12-slim
    command: python -m http.server 5000
    healthcheck:
      test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:5000')"]
      interval: 5s
      timeout: 3s
      retries: 5

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_PASSWORD: instruqt
      POSTGRES_DB: workshop
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5
COMPOSE

# Pull images first, then start
cd /root/app
docker compose pull
docker compose up -d

# Wait for all services to be healthy
docker compose wait --down-timeout 120s 2>/dev/null || \
  until [ "$(docker compose ps --format json | jq -r 'select(.Health != "" and .Health != "healthy") | .Name' | wc -l)" -eq 0 ]; do
    sleep 2
  done
```

## Config.yml Implications

Same as Docker on VM, but size up:

| Field | Recommended |
|-------|-------------|
| `machine_type` | `n1-standard-4` minimum for 3+ services |
| `nested_virtualization` | `true` (required) |
| `allow_external_ingress` | Include `high-ports` for any service port exposed to tabs |

Service tabs connect to the VM hostname on the published port. Example: a service tab with `port: 8080` reaches the Compose `web` service's published port.

## Common Pitfalls

- **Services not ready when learner arrives**. Always health-check services in setup and wait for them to pass. Without this, the first challenge may show connection errors.
- **Port conflicts**. Two Compose services cannot publish the same host port. Map each service to a distinct port (e.g., web on 8080, API on 5000, admin on 9090).
- **Compose file not found**. `docker compose up` looks for `compose.yml` or `docker-compose.yml` in the current directory. Either `cd` to the directory or use `docker compose -f /path/to/compose.yml up`.
- **Images pulled at startup**. Running `docker compose up -d` without a prior `docker compose pull` fetches images serially on first container start. Always `docker compose pull` first for parallel downloads.
- **Forgetting `-d` flag**. Without `-d`, `docker compose up` blocks the setup script and the track never starts.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Full-stack application debugging (web server + API + database + cache)
- Observability stack setup (application + Prometheus + Grafana)
- Message queue workshops (producer + broker + consumer)
- Microservices communication patterns (multiple API services + service mesh sidecar)

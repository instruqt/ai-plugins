# Single VM with Docker Compose

One virtual machine running a multi-service stack via Docker Compose. All services are orchestrated by a single `docker-compose.yml` on one host, communicating over a Compose network with service-name DNS resolution. The VM also provides systemd for anything that does not fit in a container.

See `modifiers/docker-compose-on-vm.md` for Compose setup details, health check patterns, and multi-file strategies.

## Config.yml Example

```yaml
version: "3"
virtualmachines:
  - name: workstation
    image: instruqt/docker-28-3
    machine_type: n1-standard-4
    shell: /bin/bash
    nested_virtualization: true
    allow_external_ingress:
      - http
      - https
      - high-ports
    provision_ssl_certificate: true
```

### Setup Script Pattern

Write the compose file via heredoc in the setup script, then bring the stack up:

```bash
mkdir -p /opt/app

cat <<'COMPOSE' > /opt/app/docker-compose.yml
services:
  web:
    image: nginx:1.27
    ports:
      - "8080:80"
    depends_on:
      api:
        condition: service_healthy

  api:
    image: node:22-slim
    working_dir: /app
    volumes:
      - ./api:/app
    environment:
      DATABASE_URL: postgres://app:secret@db:5432/app
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 5s
      retries: 10

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: app
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
COMPOSE

cd /opt/app && docker compose up -d
```

## When to Use

- The application is a multi-service stack (web + database + cache + worker) that the learner interacts with as a whole.
- Track teaches Docker Compose itself (writing compose files, scaling services, networking).
- The learner needs service-name DNS resolution between containers (e.g., `api` can reach `db` by hostname).
- Track needs more than two cooperating containers but does not need separate hosts.

## When NOT to Use

- Only one or two containers with no inter-service networking -- use `single-vm-docker.md` (simpler).
- Services need to run on separate hosts for the learning objective (e.g., teaching distributed deployment) -- use `multi-vm.md`.
- Track teaches Kubernetes -- use `single-vm-kind-k3s.md` instead.
- The multi-service stack is only background infrastructure and the learner never interacts with individual services -- consider whether `single-vm-docker.md` with a few `docker run` commands is simpler.

## Common Pitfalls

- **Missing `nested_virtualization: true`**. Docker on Instruqt VMs requires `/dev/kvm`. Without this flag, Docker and Compose will not start.
- **Undersized machine type**. Multiple containers sharing one VM need headroom. Use `n1-standard-4` minimum for stacks with 3+ services. Memory-heavy stacks (JVM, databases) may need `n1-standard-8`.
- **Race conditions on startup**. Use `depends_on` with `condition: service_healthy` and proper health checks. Never assume service ordering without health checks -- Compose starts containers in dependency order but does not wait for readiness by default.
- **Slow image pulls**. A stack with 4-5 images can take minutes to pull. For production tracks, pre-pull images into a custom Packer image (see `modifiers/custom-image-packer.md`).
- **Port conflicts**. Only one service can bind to a given host port. Map each service to a distinct host port (e.g., web on 8080, API on 3000, admin on 9090) and configure service tabs accordingly.
- **Compose file location**. Keep the compose file in a predictable location (e.g., `/opt/app/docker-compose.yml`) so check scripts and later challenges can find it. Document the path in a comment at the top of setup.
- **Volume persistence across challenges**. Named volumes survive `docker compose down` and persist across challenges. Use this intentionally -- if a challenge needs a clean database, drop the volume explicitly.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Docker Compose fundamentals (writing compose files, networking, volumes, scaling)
- Full-stack application workshops (web frontend + API + database + cache)
- Microservices observability (application stack + Prometheus + Grafana + Jaeger)
- CI/CD pipeline workshops (app + build server + artifact registry)
- Message queue patterns (producer + consumer + broker like RabbitMQ or Kafka)

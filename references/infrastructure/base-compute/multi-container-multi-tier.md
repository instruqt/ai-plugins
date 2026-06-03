# Multi-Container Multi-Tier

Multiple Instruqt containers forming a multi-tier application -- for example, a learner workstation plus a database and a cache. Each container is a separate entry under `containers:` in config.yml. They are Instruqt-managed containers (not Docker Compose on a VM), sharing a private sandbox network where each container's `name:` field is its hostname.

## Config.yml Example

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/shell
    ports:
      - 8080
    memory: 512
    shell: /bin/bash
    entrypoint: bash
    environment:
      DATABASE_URL: postgresql://postgres:instruqt@database:5432/app
      REDIS_URL: redis://cache:6379

  - name: database
    image: postgres:16
    ports:
      - 5432
    memory: 256
    environment:
      POSTGRES_PASSWORD: instruqt
      POSTGRES_DB: app
    private: true

  - name: cache
    image: redis:7
    ports:
      - 6379
    memory: 128
    private: true
```

### Tab configuration

Tabs target specific containers by hostname. Backend containers marked `private: true` are not directly accessible to learners.

```yaml
tabs:
  - title: Terminal
    type: terminal
    hostname: workstation
  - title: Web App
    type: service
    hostname: workstation
    port: 8080
```

### Variant: web frontend + API + database

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/shell
    ports:
      - 3000
    memory: 512
    shell: /bin/bash
    entrypoint: bash

  - name: api
    image: node:22-bookworm
    ports:
      - 4000
    memory: 512
    shell: /bin/bash
    entrypoint: bash
    environment:
      DATABASE_URL: postgresql://postgres:instruqt@database:5432/app
      PORT: "4000"

  - name: database
    image: postgres:16
    ports:
      - 5432
    memory: 256
    environment:
      POSTGRES_PASSWORD: instruqt
      POSTGRES_DB: app
    private: true
```

## Cross-Container Communication

All containers share a private sandbox network. Each container's `name` is its hostname:

- `workstation` can reach `database` at `database:5432`.
- `workstation` can reach `cache` at `cache:6379`.
- No DNS configuration needed -- Instruqt handles hostname resolution.

Pass connection strings via the `environment:` block on the container that needs them (as shown above).

## When to Use

- Track teaches interaction with a multi-service application (web + API + database + cache) and none of the services require VM features.
- Track needs a realistic service topology but not Docker itself -- learners should focus on using the services, not running them.
- Faster startup than VMs. Container-based multi-tier sandboxes are ready in seconds rather than minutes.
- Track teaches database queries, API integration, or application configuration where the services are provided, not built.

## When NOT to Use

- Track needs Docker commands (containers have no Docker socket -- use a VM pattern).
- Track needs systemd services (use a VM).
- Services require significant memory or CPU (containers share a smaller resource budget than VMs).
- Track teaches how to set up the multi-tier stack itself (Docker Compose on a VM is more appropriate).
- Track needs more than ~4-5 containers -- resource limits and complexity favor a single VM with Docker Compose.

## Common Pitfalls

- **Concurrent startup**. All containers start simultaneously. If the workstation's setup script tries to connect to the database before it is ready, it will fail. Poll in setup scripts: `until pg_isready -h database -p 5432; do sleep 2; done`.
- **Missing `ports:` entries**. A container's port must be listed under `ports:` for other containers (or tabs) to reach it. Forgetting `5432` on the database container means nothing can connect.
- **Confusing this with Docker Compose**. These are Instruqt-managed containers, not a `docker-compose.yml`. There is no `volumes:`, `networks:`, or `depends_on:`. File sharing between containers requires `agent variable` or copying via scripts.
- **No shared filesystem**. Unlike Docker Compose volumes, Instruqt containers have no shared mount. To seed the database, run SQL commands from the workstation via `psql -h database` in the setup script.
- **Memory budget**. Each container's memory is drawn from the sandbox total. Three containers at 512 MB each use 1.5 GB. Size each container for its actual needs -- Redis rarely needs more than 128 MB.
- **`private: true` means no tabs**. Marking a container as private prevents learners from opening terminal or service tabs to it. Use this for backend services the learner should not directly access.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Web application fundamentals (connect a frontend to a database via an API)
- SQL query workshops (workstation + PostgreSQL or MySQL container)
- Redis caching patterns (workstation + application container + Redis)
- API gateway configuration (workstation + multiple backend service containers)
- Application monitoring basics (workstation + app container + metrics store)

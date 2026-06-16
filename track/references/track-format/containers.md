# Containers

Docker containers defined in `config.yml` under the `containers` key. Each container becomes a hostname accessible within the sandbox network and optionally available to learners via tabs.

## Structure

```yaml
version: "3"
containers:
  - name: shell
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
      - 3000
    memory: 512
    shell: /bin/bash
    entrypoint: bash
    environment:
      NODE_ENV: development
      DB_HOST: database
    private: false

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

## Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | string | required | Unique name, becomes the hostname and is used in tab references |
| `image` | string | required | Docker image reference (registry/image:tag) |
| `ports` | list of int | — | Ports to expose within the sandbox network |
| `memory` | int (MB) | 256 | Memory allocation for the container |
| `shell` | string | `/bin/sh` | Default shell for terminal tabs connected to this container |
| `entrypoint` | string | — | Overrides the image ENTRYPOINT |
| `environment` | map | — | Key-value environment variables |
| `private` | bool | false | When `true`, container is not directly accessible to learners (no tabs). Used for backend services like databases |

## Common Images

### gcr.io/instruqt/shell

Lightweight base image. No cloud CLIs installed.

- Only `INSTRUQT_AWS_ACCOUNT_*` prefixed environment variables are available (not the unprefixed `AWS_ACCESS_KEY_ID` form)
- Good for tracks that don't need cloud provider interaction

### gcr.io/instruqt/cloud-client (and :2)

Full-featured image with AWS, Azure, and GCP CLIs pre-installed.

- Unprefixed AWS env vars available (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, etc.)
- Bootstrap script maps from `INSTRUQT_AWS_ACCOUNT_*` form to unprefixed form automatically
- Tag `:2` is the versioned variant

### Custom images

Images can come from:
- Private Artifact Registries (GCR, ECR, etc.)
- Upstream registries directly (`node:22-bookworm`, `python:3.12-slim`, `nginx:latest`, etc.)

## Entrypoint

The `entrypoint` field overrides the image's ENTRYPOINT directive. Commonly set to `"bash"` alongside `shell: /bin/bash` to ensure interactive shell access works correctly.

```yaml
- name: shell
  image: gcr.io/instruqt/shell
  shell: /bin/bash
  entrypoint: bash
```

## Private containers

Setting `private: true` marks a container as backend-only. It is reachable by other containers via its hostname but learners cannot open tabs to it.

Use for databases, message queues, API backends, and other supporting services:

```yaml
- name: redis
  image: redis:7
  ports:
    - 6379
  private: true
```

## Display server constraint

Containers do NOT support virtual display servers (X11, Xvfb). Applications that require a graphical display (KasmVNC, desktop environments) must run in a virtual machine instead.

code-server runs fine in containers because it is a web server, not a GUI application.

## Environment variables and cloud accounts

When an AWS account is configured in `config.yml`:

- **On `gcr.io/instruqt/cloud-client`**: Unprefixed AWS env vars are available (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`). The bootstrap process maps from the `INSTRUQT_AWS_ACCOUNT_*` form automatically.
- **On `gcr.io/instruqt/shell`**: Only the `INSTRUQT_AWS_ACCOUNT_{NAME}_AWS_ACCESS_KEY_ID` (and similar) prefixed form is available. You must map them manually in setup scripts if needed.

## Examples

### Single container with editor and terminal

```yaml
version: "3"
containers:
  - name: shell
    image: gcr.io/instruqt/shell
    ports:
      - 8080
    shell: /bin/bash
    entrypoint: bash
```

### Application stack with private backend

```yaml
version: "3"
containers:
  - name: app
    image: node:22-bookworm
    ports:
      - 3000
    memory: 512
    shell: /bin/bash
    entrypoint: bash
    environment:
      DATABASE_URL: postgresql://postgres:instruqt@database:5432/app
      NODE_ENV: development

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

### Cloud-enabled container

```yaml
version: "3"
containers:
  - name: shell
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 512
    shell: /bin/bash

aws_accounts:
  - name: sandbox
    services:
      - ec2
      - s3
    regions:
      - us-west-2
    managed_policies:
      - arn:aws:iam::aws:policy/ReadOnlyAccess
```

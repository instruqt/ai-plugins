# Multi-Container Sidecar

A primary Instruqt container paired with one or more helper containers that handle cross-cutting concerns: reverse proxy, TLS termination, log aggregation, metrics collection, or authentication. The sidecar and primary share the sandbox network and communicate via hostname. The sidecar typically exposes the public-facing port while the primary stays internal.

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

  - name: app
    image: node:22-bookworm
    ports:
      - 3000
    memory: 512
    shell: /bin/bash
    entrypoint: bash
    environment:
      PORT: "3000"
    private: true

  - name: proxy
    image: nginx:1.27
    ports:
      - 80
      - 443
    memory: 128
    environment:
      UPSTREAM_HOST: app
      UPSTREAM_PORT: "3000"
```

### Nginx proxy configuration

Seed the nginx config in `setup-proxy`:

```bash
#!/bin/bash

cat > /etc/nginx/conf.d/default.conf <<'CONF'
server {
    listen 80;

    location / {
        proxy_pass http://app:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
CONF

nginx -s reload
```

### Tab configuration

Expose the service tab on the sidecar's port. The learner interacts with the proxy, not the app directly:

```yaml
tabs:
  - title: Terminal
    type: terminal
    hostname: workstation
  - title: Web App
    type: service
    hostname: proxy
    port: 80
```

### Variant: metrics sidecar

```yaml
version: "3"
containers:
  - name: workstation
    image: gcr.io/instruqt/shell
    ports:
      - 8080
    memory: 256
    shell: /bin/bash
    entrypoint: bash

  - name: app
    image: node:22-bookworm
    ports:
      - 3000
      - 9090
    memory: 512
    shell: /bin/bash
    entrypoint: bash
    environment:
      METRICS_PORT: "9090"

  - name: prometheus
    image: prom/prometheus:v2.52.0
    ports:
      - 9091
    memory: 256
    environment:
      APP_METRICS_URL: "http://app:9090/metrics"

  - name: grafana
    image: grafana/grafana:11.0.0
    ports:
      - 3001
    memory: 256
    environment:
      GF_SECURITY_ADMIN_PASSWORD: instruqt
      GF_DATASOURCES_DEFAULT_URL: "http://prometheus:9091"
```

## When to Use

- Track teaches reverse proxy configuration (nginx, Envoy, HAProxy, Caddy) and the learner needs to see proxy and application as separate hosts.
- Track needs TLS termination in front of an application that does not handle TLS natively.
- Track teaches observability patterns where a metrics collector or log aggregator runs alongside an application.
- The sidecar strips or rewrites headers (e.g., Content-Security-Policy) that would otherwise break Instruqt service tabs.
- Track teaches API gateway or authentication proxy patterns (OAuth2 Proxy, Keycloak Gatekeeper).

## When NOT to Use

- The proxy or helper can run inside the primary container (e.g., installing nginx in the same container). If the learner does not need to inspect the sidecar independently, a single container with both processes is simpler.
- The sidecar needs Docker, systemd, or kernel features -- use a VM pattern.
- The architecture has many services beyond just primary + sidecar -- the `multi-container-multi-tier.md` pattern is a better fit.

## Sidecar vs Configuring the Primary Container

Use a sidecar when:
- The sidecar is a standard off-the-shelf image (nginx, Envoy, Prometheus) that should not be modified.
- The learner needs to interact with both the sidecar and primary independently (e.g., editing nginx config, viewing proxy logs).
- Separation of concerns matters for the lesson (teaching why reverse proxies exist).

Configure the primary container instead when:
- The helper process is lightweight and does not need its own image.
- The learner should not see or care about the helper.
- Adding a second container would increase complexity without educational value.

## Common Pitfalls

- **Port routing confusion**. The service tab should point at the sidecar (the public-facing port), not the primary container. If the tab targets the app directly, the sidecar is bypassed and the lesson breaks.
- **Startup ordering**. The sidecar may start before the primary is listening. Nginx returns 502 if the upstream is not yet available. Either configure nginx to retry upstream connections, or poll in `setup-proxy`: `until nc -z app 3000; do sleep 1; done`.
- **Forgetting `ports:` on the primary**. Even though the primary is `private: true`, its port must be listed under `ports:` for the sidecar to reach it via hostname.
- **Config file seeding**. Sidecar images like nginx use default configs. You must overwrite the config in the sidecar's setup script. Forgetting this means nginx serves its default welcome page instead of proxying.
- **CSP header stripping**. A common reason for the sidecar pattern is to strip Content-Security-Policy headers that prevent Instruqt service tabs from rendering. Add `proxy_hide_header Content-Security-Policy;` and `proxy_hide_header X-Frame-Options;` to the nginx config if needed.
- **Memory overhead**. Sidecars like nginx or HAProxy are lightweight (64-128 MB). Monitoring stacks (Prometheus + Grafana) are heavier (256 MB each). Size accordingly.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Reverse proxy fundamentals (configure nginx to proxy to a backend application)
- TLS termination workshop (Caddy or nginx with Let's Encrypt or self-signed certs)
- API gateway patterns (Envoy, Kong, or Traefik in front of microservices)
- Application monitoring (Prometheus + Grafana sidecar collecting metrics from an app)
- Authentication proxy setup (OAuth2 Proxy in front of an unprotected application)

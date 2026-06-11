# Service Tab Design

Evaluates whether service tabs correctly expose web UIs within the Instruqt sandbox, including hostname/port mapping, deep linking, and Content-Security-Policy handling for iframe embedding.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Service tabs fail to load -- wrong port, missing hostname, or CSP blocks rendering with no workaround |
| 2 | Below Standard | Service tab loads but shows a blank iframe or partial content due to unhandled CSP/X-Frame-Options headers |
| 3 | Adequate | Web UI is accessible but requires the learner to navigate manually; no path or deep linking configured |
| 4 | Good | All web UIs are accessible via service tabs; CSP and X-Frame-Options issues are handled; path is set where appropriate (production baseline) |
| 5 | Excellent | Service tabs create a seamless embedded experience -- deep links land on the exact page, CSP overrides are minimal and targeted, no learner friction |

## Guidance

Service tabs embed a web UI running inside the sandbox into the Instruqt player via an iframe. The tab connects to a hostname and port.

Basic service tab:

```yaml
- title: Grafana
  type: service
  hostname: workstation
  port: 3000
```

Use `path` to deep-link the learner directly to the relevant page:

```yaml
- title: Dashboard
  type: service
  hostname: workstation
  port: 3000
  path: /d/main/overview?orgId=1
```

Many web UIs set `X-Frame-Options` or `Content-Security-Policy` headers that block iframe embedding. Three approaches exist to handle this (see also the `csp-header-overrides` reference):

**Approach 1 -- Service tab header overrides** (simplest when it works):

```yaml
- title: ArgoCD
  type: service
  hostname: workstation
  port: 8080
  custom_request_headers:
    - "Origin: "
  custom_response_headers:
    - "Content-Security-Policy: "
    - "X-Frame-Options: "
```

**Approach 2 -- Reverse proxy** (when header overrides are not sufficient):

Set up Caddy or nginx in the setup script to strip headers at the proxy level. The service tab then points to the proxy port.

**Approach 3 -- new_window** (last resort for truly non-iframable UIs):

```yaml
- title: External UI
  type: service
  hostname: workstation
  port: 8443
  new_window: true
```

This opens the UI in a separate browser tab instead of embedding it.

Good -- service tab with deep link and CSP override:

```yaml
- title: Vault UI
  type: service
  hostname: workstation
  port: 8200
  path: /ui/vault/secrets
  custom_response_headers:
    - "Content-Security-Policy: "
    - "X-Frame-Options: "
```

Good -- new_window for a UI that cannot be embedded:

```yaml
- title: Cloud Console
  type: service
  hostname: workstation
  port: 8080
  new_window: true
```

Bad -- no CSP handling when the UI sets X-Frame-Options DENY:

```yaml
- title: Jenkins
  type: service
  hostname: workstation
  port: 8080
```

Bad -- wrong port (application listens on 3000, tab points to 80):

```yaml
- title: App
  type: service
  hostname: workstation
  port: 80
```

Bad -- using HTTPS port without the application actually serving TLS inside the sandbox:

```yaml
- title: Dashboard
  type: service
  hostname: workstation
  port: 443
```

## What to Watch For

- The port must match what the application actually listens on inside the container/VM, not a standard port assumption
- `path` values must include the leading slash and any required query parameters
- CSP and X-Frame-Options blocking is the most common cause of blank service tabs -- always test in the player, not just by curling the port
- `custom_response_headers` with an empty value (e.g., `"X-Frame-Options: "`) strips the header; setting a value replaces it
- `custom_request_headers` can strip Origin headers that trigger CORS rejections in some UIs
- `new_window: true` breaks the seamless experience -- only use it when iframe embedding is truly impossible
- If the web UI takes time to start, the service tab will show an error until it is ready; coordinate with setup script timing
- socat port forwarding does NOT help with CSP/X-Frame-Options -- it forwards headers unchanged
- **Co-hosted API + portal on same hostname/port** -- some applications serve both a web UI and an API on the same port (e.g., Grafana serves the dashboard UI and `/api/` endpoints on port 3000). A single service tab handles both:

```yaml
- title: Grafana
  type: service
  hostname: workstation
  port: 3000
  path: /d/main/overview
```

Do not create separate service tabs for the API and UI when they share a port — use one tab for the UI and let the learner access the API from the terminal via curl. If you need to show API responses in a tab, use a terminal tab with a `cmd:` that runs the curl command.

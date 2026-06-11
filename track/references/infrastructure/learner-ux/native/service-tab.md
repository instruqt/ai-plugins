# Service Tab

Embeds a web UI running inside the sandbox into the Instruqt player via an iframe proxy. The simplest way to expose any HTTP service to the learner without leaving the track interface.

## When to Use

- The sandbox runs a web server (dashboard, admin panel, API explorer, custom application) and the learner needs to interact with it.
- The application renders correctly inside an iframe (no popup windows, no OAuth redirects, no JavaScript that checks `window.top`).
- You want the learner to stay in the Instruqt player rather than opening a separate browser window.

## Config.yml Example

Basic service tab (web UI on port 8080):

```yaml
tabs:
  - title: Web App
    type: service
    hostname: workstation
    port: 8080
```

Deep-linked to a specific page:

```yaml
tabs:
  - title: API Docs
    type: service
    hostname: workstation
    port: 3000
    path: /swagger
```

With CSP header override (the most common fix for blank iframes):

```yaml
tabs:
  - title: Dashboard
    type: service
    hostname: workstation
    port: 9090
    custom_response_headers:
      Content-Security-Policy: ""
      X-Frame-Options: ""
```

## Pairs With

Any base compute pattern that runs a web server -- VMs with Docker, containers exposing HTTP ports, Kubernetes with port-forwarded services. The VM or container must have `allow_external_ingress` configured with `http`, `https`, and `high-ports` for the Instruqt proxy to reach the port.

## Common Patterns

- **Terminal + Service tab.** Learner makes changes via CLI, observes results in the embedded web UI. The bread-and-butter layout for infrastructure and DevOps tracks.
- **Multiple service tabs.** Tracks teaching microservices or monitoring stacks may expose several web UIs (the app, Grafana, Prometheus) each on its own service tab.
- **Path for deep linking.** Use `path:` to land the learner on the exact page relevant to the current challenge, not the application's root.

## Common Pitfalls

- **CSP / X-Frame-Options blocking the iframe.** This is the number one issue with service tabs. Many web frameworks ship with `Content-Security-Policy: frame-ancestors 'self'` or `X-Frame-Options: DENY` by default. The iframe will be blank or show a browser error. Fix with `custom_response_headers` to strip the offending headers, or configure a reverse proxy (Caddy, nginx) in front of the application.
- **Service not ready when tab loads.** If the web server starts slowly, the learner sees a connection error. Use setup scripts to wait for the port before finishing: `while ! curl -sf http://localhost:8080 > /dev/null; do sleep 1; done`.
- **Wrong port.** The `port:` field must match the port the application is listening on inside the sandbox, not a mapped or external port.
- **HTTPS-only applications.** Some applications redirect HTTP to HTTPS or require TLS. The Instruqt proxy handles TLS termination, so the service tab should point to the HTTP port. If the application insists on HTTPS, ensure `provision_ssl_certificate: true` is set on the VM.

## When NOT to Use

- **External URLs.** For public websites, documentation, or SaaS dashboards not running inside the sandbox, use `type: website` or `type: external` instead.
- **Cloud consoles (AWS, Azure, GCP).** These require a full browser context with cookie handling, popups, and OAuth. Use a virtual browser tab instead.
- **JavaScript-heavy SPAs that break in iframes.** Applications that check `window.top !== window.self`, open popup windows, or perform OAuth redirects will not work in a service tab iframe. Use a virtual browser pointing at the sandbox service instead.
- **Applications that need the browser's address bar.** The learner cannot see or modify the URL in a service tab. If the exercise requires inspecting or navigating URLs manually, use a virtual browser.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Monitoring stack walkthrough with Grafana and Prometheus dashboards
- Web application deployment where learners observe their changes in the running app
- API development workshop with Swagger UI for testing endpoints
- Container orchestration lab with a Kubernetes dashboard

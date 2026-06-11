# Port Exposure

Evaluates whether port configuration and external ingress settings correctly expose the services learners need while minimizing unnecessary exposure.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No port exposure configured when the track requires web access, or allow_external_ingress missing entirely |
| 2 | Below Standard | Ports exposed but mismatched with service tabs -- tabs reference ports that are not open, or ports open with no corresponding tab |
| 3 | Adequate | Core ports exposed and tabs mostly aligned, but SSL not provisioned for HTTPS services or unnecessary ports left open |
| 4 | Good | All needed ports exposed; SSL provisioned where HTTPS is required; port numbers align with service tab definitions (production baseline) |
| 5 | Excellent | Minimal port exposure -- only the ports the learner actually needs are open; no leftover or speculative ports; SSL and ingress precisely scoped |

## Guidance

The `allow_external_ingress` field on a VM or container controls which port ranges are accessible from the Instruqt platform (and therefore from learner browser tabs). The standard set covers most web application scenarios:

- **http** (port 80) -- plain HTTP services
- **https** (port 443) -- TLS-terminated services
- **high-ports** (ports 1024-65535) -- application servers, Node.js, Flask, custom APIs, dashboards

For web-facing services, pair `allow_external_ingress` with `provision_ssl_certificate: true` so the Instruqt proxy can terminate TLS and the learner sees a secure connection.

Good -- web application with correct port exposure:

```yaml
virtualmachines:
- name: workstation
  machine_type: n1-standard-2
  image: ubuntu-22-04
  allow_external_ingress:
  - http
  - https
  - high-ports
  provision_ssl_certificate: true
```

Good -- minimal exposure for an API-only service:

```yaml
containers:
- name: api
  image: node:22
  allow_external_ingress:
  - high-ports
  provision_ssl_certificate: true
```

Bad -- HTTPS tab configured but no SSL certificate provisioned:

```yaml
virtualmachines:
- name: workstation
  machine_type: n1-standard-2
  image: ubuntu-22-04
  allow_external_ingress:
  - https
  # Missing: provision_ssl_certificate: true
```

Bad -- no ingress configured but track has a service tab:

```yaml
# Tab expects port 8080 but VM has no allow_external_ingress
virtualmachines:
- name: workstation
  machine_type: n1-standard-2
  image: ubuntu-22-04
```

Bad -- overly broad exposure when only one port is needed:

```yaml
# Only running a static file server on port 8080
# but exposing http, https, and all high ports
virtualmachines:
- name: workstation
  machine_type: n1-standard-2
  image: ubuntu-22-04
  allow_external_ingress:
  - http
  - https
  - high-ports
```

## What to Watch For

- Service tabs that reference ports on a VM or container with no `allow_external_ingress` -- the tab will show a connection error
- `provision_ssl_certificate: true` missing when the service tab uses HTTPS -- learners see certificate warnings or mixed content errors
- Ports open with no corresponding tab or learner-facing purpose -- unnecessary attack surface
- The `[http, https, high-ports]` triple is correct for most web app tracks but excessive for tracks that only use terminal tabs
- Container ports behave differently from VM ports -- containers expose the port their process listens on, while VMs require the service to be running and bound

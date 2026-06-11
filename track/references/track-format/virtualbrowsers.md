# Virtual Browsers

Virtual browser tabs render a full browser inside the Instruqt UI, typically used for cloud console sign-in pages. Defined in `config.yml` under the `virtualbrowsers:` key.

## Structure

```yaml
virtualbrowsers:
  - name: <hostname>
    url: <target-url>
```

The `name` field becomes a tab target in challenge `assignment.md` frontmatter via `type: browser`.

## Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Logical hostname for the browser instance. Referenced by `type: browser` tabs. |
| `url` | string | yes | URL the browser navigates to on load. Supports interpolation and internal hostnames. |

### URL Interpolation

- `${_SANDBOX_ID}` -- replaced at runtime with the sandbox identifier.
- Internal VM hostnames resolve automatically through the Instruqt proxy. The URL can target a VM by its hostname directly (e.g., `https://host01:8443/app`).
- The pattern `<hostname>.<sandbox-id>.instruqt.io` works in the `url` field. The hostname portion must match a `name:` field in `config.yml`.

### SSL for Internal Hostnames

When pointing a virtual browser at an internal VM over HTTPS, pair the VM definition with `provision_ssl_certificate: true` so Instruqt provisions a valid TLS certificate for the hostname.

## Examples

### AWS Console Sign-In

```yaml
virtualbrowsers:
  - name: aws-console
    url: https://console.aws.amazon.com
```

### Azure Portal

```yaml
virtualbrowsers:
  - name: azure-portal
    url: https://portal.azure.com/
```

### Internal VM Web UI

```yaml
virtualmachines:
  - name: dataserver
    image: ubuntu-2204
    machine_type: n1-standard-2
    provision_ssl_certificate: true

virtualbrowsers:
  - name: dataserver
    url: https://dataserver:8443/DataAssurance
```

The Instruqt proxy resolves `dataserver` to the VM. The `provision_ssl_certificate: true` on the VM ensures the HTTPS connection succeeds.

### Sandbox-ID URL Pattern

```yaml
virtualbrowsers:
  - name: webapp
    url: https://webapp.${_SANDBOX_ID}.instruqt.io
```

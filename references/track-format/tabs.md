# Tabs

Tabs define the interactive panels shown to the learner in each challenge. Configured in the `tabs:` array of the challenge `assignment.md` frontmatter.

## Structure

```yaml
tabs:
  - title: <display-name>
    type: <tab-type>
    hostname: <target-host>
    # ... type-specific fields
```

## Tab Types

### terminal

Shell access to a host.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | yes | Display name |
| `type` | string | yes | `terminal` |
| `hostname` | string | yes | Target host |
| `cmd` | string | no | Startup command (e.g., `su - user`, `zsh`, `bash -c "npm start"`, `screen -xRR`) |
| `workdir` | string | no | Initial working directory |

```yaml
- title: Terminal
  type: terminal
  hostname: host01
  cmd: su - developer
  workdir: /home/developer/app
```

### code

File editor panel. Can open a single file or a directory tree explorer.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | yes | Display name |
| `type` | string | yes | `code` |
| `hostname` | string | yes | Target host |
| `path` | string | yes | File path or directory path for tree explorer |
| `workdir` | string | no | Working directory for the editor |

```yaml
- title: Editor
  type: code
  hostname: host01
  path: /root/app
```

**Companion code tab pair pattern** -- two code tabs, one editable and one read-only reference:

```yaml
- title: Editor
  type: code
  hostname: host01
  path: /root/workspace
- title: Reference
  type: code
  hostname: host01
  path: /root/reference
```

### service

Web UI rendered in an iframe, mediated by the Instruqt proxy. Preferred for in-sandbox web UIs.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | yes | Display name |
| `type` | string | yes | `service` |
| `hostname` | string | yes | Target host |
| `port` | integer | yes | Port number |
| `path` | string | no | URL path appended after host:port |
| `new_window` | boolean | no | Open in a new browser window |
| `custom_request_headers` | map | no | Headers added to proxied requests (CSP override) |
| `custom_response_headers` | map | no | Headers added to proxied responses (CSP override) |

```yaml
- title: Grafana
  type: service
  hostname: host01
  port: 3000
  path: /dashboards
```

**CSP override example** -- strip restrictive headers so the iframe renders:

```yaml
- title: Web App
  type: service
  hostname: host01
  port: 8080
  custom_response_headers:
    Content-Security-Policy: ""
    X-Frame-Options: ""
```

Alternatively, use a Caddy or nginx reverse proxy in front of the application to strip CSP headers before they reach the iframe.

### external

External URL tab.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | yes | Display name |
| `type` | string | yes | `external` |
| `url` | string | yes | Target URL |
| `new_window` | boolean | no | Open in a new browser window |

```yaml
- title: Documentation
  type: external
  url: https://docs.example.com/guide
```

### browser

Cloud console browser tab tied to a `virtualbrowsers` entry in `config.yml`.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | yes | Display name |
| `type` | string | yes | `browser` |
| `hostname` | string | yes | Must match a `virtualbrowsers` `name` in `config.yml` |

```yaml
- title: AWS Console
  type: browser
  hostname: aws-console
```

### website

External URL tab with explicit HTTPS URL. Use for sandbox-public URLs or SaaS forms.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | yes | Display name |
| `type` | string | yes | `website` |
| `url` | string | yes | Target URL (must be HTTPS) |
| `new_window` | boolean | no | Open in a new browser window |

Supports interpolation:
- `${_SANDBOX_ID}` -- replaced with sandbox identifier
- `${INSTRUQT_PARTICIPANT_ID}` -- replaced with participant identifier

```yaml
- title: Lab Portal
  type: website
  url: https://portal.example.com/lab/${_SANDBOX_ID}
```

## service vs website

| Aspect | `service` | `website` |
|--------|-----------|-----------|
| Mechanism | Iframe mediated by Instruqt proxy | Iframe with direct HTTPS URL |
| URL source | Constructed from hostname + port + path | Explicit `url:` field |
| Best for | In-sandbox web UIs | Sandbox-public URLs, SaaS dashboards |
| CSP control | `custom_request_headers` / `custom_response_headers` | None (URL must serve permissive headers) |

Prefer `type: service` for applications running inside the sandbox. Use `type: website` for externally accessible URLs.

## Examples

### Full Challenge Tab Set

```yaml
tabs:
  - title: Terminal
    type: terminal
    hostname: host01
  - title: Editor
    type: code
    hostname: host01
    path: /root/app
  - title: Web UI
    type: service
    hostname: host01
    port: 8080
  - title: AWS Console
    type: browser
    hostname: aws-console
  - title: Docs
    type: website
    url: https://docs.example.com
    new_window: true
```

### Terminal with Custom Shell

```yaml
tabs:
  - title: Node REPL
    type: terminal
    hostname: host01
    cmd: bash -c "cd /root/app && node"
  - title: Multiplexer
    type: terminal
    hostname: host01
    cmd: screen -xRR
```

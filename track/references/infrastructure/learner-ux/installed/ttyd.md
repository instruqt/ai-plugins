# ttyd

Browser-streamed terminal application via ttyd. Runs a terminal command and exposes it as a web page, letting learners view or interact with TUI (text user interface) applications through a service tab. Lighter than code-server when you only need terminal output in a browser frame.

## How It Differs from Terminal Tabs

| Aspect | Terminal tab | ttyd service tab |
|--------|-------------|------------------|
| Protocol | Instruqt native terminal | HTTP/WebSocket served by ttyd |
| Use case | General shell access | Streaming a specific TUI app (htop, k9s, watch) |
| Layout | Standard terminal pane | Embeddable in a service tab alongside other tabs |
| Interaction | Full shell | Can be read-only or interactive |
| Overhead | None (built-in) | ttyd process on the host |

Use ttyd when you want to stream a specific application's output into its own browser pane — for example, a live dashboard (k9s, htop, glances), a log tail, or a command output that learners should observe while working in a separate terminal tab.

For general shell access, use a standard terminal tab instead.

## Requirements

- A VM (ttyd needs to be installed; not available in most containers).
- `allow_external_ingress` with `high-ports` (ttyd runs on a custom port).
- `provision_ssl_certificate: true` for HTTPS access via service tab.
- Exposed via a service tab pointing at the ttyd port.

## Config.yml Example

```yaml
version: "3"
virtualmachines:
  - name: workstation
    image: ubuntu-os-cloud/ubuntu-2404-lts-amd64
    machine_type: n1-standard-2
    shell: /bin/bash
    allow_external_ingress:
      - http
      - https
      - high-ports
    nested_virtualization: true
    provision_ssl_certificate: true
```

## Setup

### track_scripts/setup-workstation

```bash
#!/bin/bash
set -euxo pipefail

until [ -f /opt/instruqt/bootstrap/host-bootstrap-completed ]; do
  sleep 1
done

export DEBIAN_FRONTEND=noninteractive

# Install ttyd
TTYD_VERSION="1.7.7"
ARCH=$(dpkg --print-architecture)
curl -fsSL "https://github.com/tsl0922/ttyd/releases/download/${TTYD_VERSION}/ttyd.${ARCH}" \
  -o /usr/local/bin/ttyd
chmod +x /usr/local/bin/ttyd
```

### Streaming a TUI application

Create a systemd service that runs ttyd with the target command:

```bash
# Stream htop on port 7681
cat > /etc/systemd/system/ttyd-htop.service <<'EOF'
[Unit]
Description=ttyd streaming htop
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/ttyd --port 7681 --readonly htop
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now ttyd-htop
```

### Interactive terminal via ttyd

For an interactive session (learner can type), omit `--readonly`:

```bash
cat > /etc/systemd/system/ttyd-bash.service <<'EOF'
[Unit]
Description=ttyd interactive terminal
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/ttyd --port 7682 --credential user:password bash
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now ttyd-bash
```

### Service tab configuration

```yaml
tabs:
  - title: System Monitor
    type: service
    hostname: workstation
    port: 7681

  - title: Interactive Shell
    type: service
    hostname: workstation
    port: 7682
```

## Common Use Cases

- **k9s dashboard** — `ttyd --port 7681 --readonly k9s` to stream a Kubernetes TUI dashboard while learners work in a terminal tab
- **Log streaming** — `ttyd --port 7681 --readonly tail -f /var/log/app.log` for live log observation
- **Watch commands** — `ttyd --port 7681 --readonly watch -n 2 kubectl get pods -A` for live resource monitoring
- **glances/htop** — System monitoring dashboards viewable in a browser pane
- **Custom REPL** — `ttyd --port 7681 python3` for an interactive Python session in its own tab

## ttyd Options Reference

| Flag | Description |
|------|-------------|
| `--port PORT` | Listen port (default: 7681) |
| `--readonly` | Disable input — view-only mode |
| `--credential USER:PASS` | Basic auth (omit for no auth) |
| `--interface IP` | Bind interface (default: 0.0.0.0) |
| `--max-clients N` | Maximum concurrent connections |
| `--once` | Exit after the first client disconnects |
| `--ping-interval SEC` | WebSocket keepalive interval |

## Common Pitfalls

- **Port not reachable.** Ensure `allow_external_ingress` includes `high-ports`. Without it, the service tab will not connect.
- **ttyd exits when the command exits.** If the streamed command crashes, ttyd stops. Use `Restart=on-failure` in the systemd service and consider wrapping the command in a retry loop.
- **Read-only vs interactive.** Default is interactive — learners can type into the ttyd session. For observation-only use cases (dashboards, logs), always add `--readonly` to prevent learners from accidentally closing or modifying the running application.
- **Multiple ttyd instances.** Each needs its own port. Use different ports (7681, 7682, ...) and a separate systemd service for each.
- **CSP/X-Frame-Options.** ttyd's built-in web server does not set restrictive CSP headers, so it usually works in service tabs without overrides.
- **Authentication.** Without `--credential`, anyone with the URL can access the terminal. In Instruqt sandboxes this is acceptable (sandbox is isolated), but add credentials if the track exposes the port externally.

## Example Tracks

*These examples are illustrative, not prescriptive — adapt to your specific needs.*

- Kubernetes operations workshops with live k9s monitoring pane
- Performance analysis tracks with htop/glances observation
- CI/CD pipeline tracks with live log streaming
- Database administration with live query monitoring

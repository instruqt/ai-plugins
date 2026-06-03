# Code-Server

VS Code in the browser via code-server. Provides a full IDE experience for code editing, terminal access, and extension support without requiring learners to install anything locally. Requires a VM base and is exposed via a service tab.

## Requirements

- A VM with at least `n1-standard-2` (code-server is lightweight but extensions and language servers add up).
- `allow_external_ingress` with `high-ports` (code-server runs on port 8443).
- `provision_ssl_certificate: true` for HTTPS access.
- Exposed via a service tab pointing at port 8443.

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

# Install code-server
curl -fsSL https://code-server.dev/install.sh | sh

# Create workspace directory
mkdir -p /workspace

# Create systemd service for code-server
cat > /etc/systemd/system/code-server.service <<'EOF'
[Unit]
Description=code-server
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/code-server \
  --host 0.0.0.0 \
  --port 8443 \
  --cert \
  --auth none \
  /workspace
Restart=on-failure
Environment=HOME=/root

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable code-server
systemctl start code-server

# Configure VS Code settings
mkdir -p /root/.local/share/code-server/User
cat > /root/.local/share/code-server/User/settings.json <<'EOF'
{
  "workbench.colorTheme": "Default Dark+",
  "security.workspace.trust.enabled": false,
  "terminal.integrated.defaultProfile.linux": "bash"
}
EOF

# Wait for code-server to be ready
until curl -sf https://localhost:8443 -k; do
  sleep 2
done
```

### Service tab configuration

```yaml
tabs:
  - title: Editor
    type: service
    hostname: workstation
    port: 8443
```

## When to Use

- Learner needs a full IDE with extensions, integrated terminal, and multi-file navigation.
- Track involves writing or editing code across multiple files (application development, Terraform modules, Helm charts).
- The native editor tab is too limited for the task.

See `decision-frameworks/editor-selection.md` for a comparison with the native editor tab.

## Installing Extensions

Pre-install extensions in the setup script:

```bash
code-server --install-extension ms-python.python
code-server --install-extension hashicorp.terraform
code-server --install-extension redhat.vscode-yaml

# Restart to pick up extensions
systemctl restart code-server
```

## Common Pitfalls

- **Port 8443 not reachable**. Ensure `allow_external_ingress` includes `high-ports`. Without it, the service tab will not connect.
- **Workspace trust dialog**. Without `"security.workspace.trust.enabled": false`, learners see a trust prompt on first open that confuses the flow.
- **Theme flicker**. If settings.json is written after code-server starts, the theme may flash from light to dark. Write settings before starting the service, or restart it after writing.
- **Extension installation timing**. Extensions installed after code-server starts require a restart. Install them before `systemctl start code-server` or restart afterward.
- **Self-signed certificate warning**. The `--cert` flag generates a self-signed cert. Instruqt's service tab proxy handles TLS termination, so learners do not see warnings in the embedded tab.
- **Non-root terminal profile**. By default, code-server's integrated terminal runs as root. If the track needs a different user, configure `terminal.integrated.profiles.linux` explicitly.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Python application development workshops
- Terraform/OpenTofu configuration authoring
- Node.js / React application building
- YAML/Kubernetes manifest editing
- Any track where learners write or edit code files

# noVNC

Lightweight browser-based VNC client running directly on a VM without a separate Guacamole container. noVNC is a JavaScript VNC client served over HTTP — it connects to a VNC server on the same host and exposes a web interface.

## How It Differs from Guacamole VNC

| Aspect | Guacamole VNC | noVNC |
|--------|--------------|-------|
| Architecture | Separate Guacamole container + VM | Single VM, noVNC runs alongside VNC server |
| Complexity | More components (container + user-mapping.xml) | Simpler (one process, no container needed) |
| Features | Multi-protocol (RDP, VNC, SSH), auth, session recording | VNC only, minimal auth |
| Resource overhead | Extra container (~512 MB) | Minimal (lightweight web server) |
| Best for | Production tracks, multiple protocols | Quick prototyping, simple GUI access |

## Config.yml Example

```yaml
version: "3"
virtualmachines:
  - name: workstation
    image: ubuntu-os-cloud/ubuntu-2404-lts-amd64
    machine_type: n1-standard-4
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

apt-get update
apt-get install -y xfce4 xfce4-goodies tigervnc-standalone-server novnc websockify

mkdir -p /root/.vnc

# Set VNC password
echo "instruqt" | vncpasswd -f > /root/.vnc/passwd
chmod 600 /root/.vnc/passwd

# Configure VNC to start xfce
cat > /root/.vnc/xstartup <<'EOF'
#!/bin/bash
xfce4-session &
EOF
chmod +x /root/.vnc/xstartup

# Start VNC server on display :1 (port 5901)
vncserver :1 -geometry 1920x1080 -depth 24

# Start noVNC (websockify bridges WebSocket to VNC)
websockify --web /usr/share/novnc 8080 localhost:5901 &

until curl -sf http://localhost:8080 -o /dev/null; do
  sleep 2
done
```

### Service tab configuration

```yaml
tabs:
  - title: Desktop
    type: service
    hostname: workstation
    port: 8080
    path: /vnc.html?autoconnect=true&password=instruqt
```

## When to Use

- Quick GUI access without the overhead of a Guacamole container.
- Track prototype or proof-of-concept where simplicity matters.
- Only VNC is needed (no RDP, no SSH-over-browser).

## When NOT to Use

- Production tracks with polish requirements — Guacamole provides a better UX (auto-resize, clipboard, session management).
- Track needs RDP — noVNC is VNC only.
- Track needs multiple protocol support — use Guacamole.

## Common Pitfalls

- **websockify must stay running**. It bridges WebSocket traffic to the VNC socket. If it dies, the noVNC page shows a connection error. Use `nohup` or a systemd service to keep it alive.
- **VNC password in URL**. The `password=instruqt` query parameter is visible in the service tab URL. This is acceptable for Instruqt sandboxes (single-learner, ephemeral) but should not be used with sensitive credentials.
- **Auto-connect parameter**. Without `autoconnect=true` in the path, the learner sees a "Connect" button before reaching the desktop. Always include it for a seamless experience.
- **Resolution mismatch**. Set the VNC geometry (`-geometry 1920x1080`) to match the expected browser viewport. noVNC does not auto-resize like Guacamole's `resize-method: display-update`.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Quick proof-of-concept for GUI-based labs
- VNC server configuration and troubleshooting tutorials
- Lightweight graphical desktop for tools that have no web UI

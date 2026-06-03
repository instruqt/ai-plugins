# Guacamole VNC

A VM paired with a Guacamole container to provide browser-based GUI access via the VNC protocol. Use when the target system exposes VNC rather than RDP, or when RDP is not available (headless Linux servers with VNC, macOS remote access, embedded systems).

## How It Differs from Guacamole RDP

| Aspect | RDP | VNC |
|--------|-----|-----|
| Protocol | RDP (port 3389) | VNC (port 5900+) |
| Session type | Full Windows or Linux login session | Screen-sharing of an existing X display |
| Multi-user | Yes (concurrent sessions) | No (shares one display) |
| Clipboard | Built-in bidirectional | Varies by VNC server |
| Best for | Windows desktops, Linux with xrdp | Headless Linux with VNC server, existing X sessions |

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

containers:
  - name: guacamole
    image: gcr.io/instruqt/guacamole
    ports:
      - 8080
    memory: 512
```

## Setup

### track_scripts/setup-workstation (install desktop + VNC server)

```bash
#!/bin/bash

apt-get update
apt-get install -y xfce4 xfce4-goodies tigervnc-standalone-server

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
```

### track_scripts/setup-guacamole (write user-mapping for VNC)

```bash
#!/bin/bash

mkdir -p /config/guacamole

cat > /config/guacamole/user-mapping.xml <<'GUAC'
<user-mapping>
  <authorize username="instruqt" password="instruqt">
    <connection name="Desktop">
      <protocol>vnc</protocol>
      <param name="hostname">workstation</param>
      <param name="port">5901</param>
      <param name="password">instruqt</param>
    </connection>
  </authorize>
</user-mapping>
GUAC

until nc -z workstation 5901; do
  echo "Waiting for VNC on workstation:5901..."
  sleep 2
done
```

### Service tab configuration

```yaml
tabs:
  - title: Desktop
    type: service
    hostname: guacamole
    port: 8080
    path: /#/client/Desktop
```

## When to Use

- The target system only exposes VNC (no RDP available).
- Track demonstrates VNC-specific workflows (remote access to headless servers, embedded systems).
- An existing application already runs on a VNC display.

## When NOT to Use

- Windows desktop — use Guacamole RDP instead (native RDP support).
- Just need a GUI app in a browser — KasmVNC is simpler (no separate Guacamole container).
- Need high performance rendering — RDP generally performs better than VNC for complex UIs.

## Common Pitfalls

- **VNC port numbering**. Display `:1` is port 5901, `:2` is 5902, etc. Match the port in the Guacamole config to the display number.
- **VNC server not started**. Unlike xrdp (which is a systemd service), `vncserver` is often started manually. Ensure it runs in the setup script and poll with `nc -z workstation 5901`.
- **Single display**. VNC shares one display. Multiple learners cannot have independent sessions (not relevant for Instruqt's single-learner sandbox, but matters for documentation accuracy).
- **Clipboard support**. VNC clipboard sharing varies by server. TigerVNC supports it; some lightweight servers do not. Test copy-paste during track development.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Remote access to headless Linux servers
- SAML/SSO configuration requiring a browser on the server
- Application monitoring with GUI tools running on the target host
- Legacy system administration via VNC

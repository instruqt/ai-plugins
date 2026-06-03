# Guacamole RDP

A VM paired with a Guacamole container to provide browser-based GUI desktop access via RDP. Use for Windows application demos, Linux desktop environments, or any track that needs a full graphical desktop.

## Requirements

- A VM with at least `n1-standard-4` (GUI desktops are resource-intensive).
- A Guacamole container (`gcr.io/instruqt/guacamole`) on port 8080.
- xrdp installed and running on the VM.
- Exposed via a service tab pointing at the Guacamole container.

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

### track_scripts/setup-workstation (install desktop + xrdp)

```bash
#!/bin/bash

# Install desktop environment and xrdp
apt-get update
apt-get install -y xfce4 xfce4-goodies xrdp

# Configure xrdp to use xfce
echo "xfce4-session" > /root/.xsession

# Set a password for the RDP user
echo "root:instruqt" | chpasswd

systemctl enable xrdp
systemctl start xrdp
```

### track_scripts/setup-guacamole (write user-mapping)

```bash
#!/bin/bash

mkdir -p /config/guacamole

cat > /config/guacamole/user-mapping.xml <<'GUAC'
<user-mapping>
  <authorize username="instruqt" password="instruqt">
    <connection name="Desktop">
      <protocol>rdp</protocol>
      <param name="hostname">workstation</param>
      <param name="port">3389</param>
      <param name="username">root</param>
      <param name="password">instruqt</param>
      <param name="ignore-cert">true</param>
      <param name="security">any</param>
      <param name="resize-method">display-update</param>
    </connection>
  </authorize>
</user-mapping>
GUAC

until nc -z workstation 3389; do
  echo "Waiting for RDP on workstation:3389..."
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

- Track requires a Windows desktop (Windows Server administration, .NET development).
- Track needs a full Linux desktop environment (GUI tools, file managers, browsers).
- Learner needs to interact with a graphical application that has no web UI.

See `decision-frameworks/gui-access-method.md` for a comparison with VNC, KasmVNC, and noVNC.

## Common Pitfalls

- **RDP not ready**. The Guacamole setup script must poll `nc -z workstation 3389` before completing. If Guacamole starts before xrdp is listening, the connection fails.
- **Password with XML special characters**. If the RDP password contains `<`, `>`, `&`, `"`, or `'`, the user-mapping.xml will be malformed. Use alphanumeric passwords.
- **Guacamole v1 vs v2 path format**. V1 uses `/#/client/<ConnectionName>`. V2 uses base64-encoded connection strings. Check the image version.
- **Desktop environment weight**. Full GNOME/KDE desktops are heavy. XFCE is recommended. Size the VM to at least `n1-standard-4`.
- **Screen resolution**. Use `resize-method: display-update` in the Guacamole config to auto-resize to the browser window.
- **Clipboard sharing**. Copy-paste between the learner's machine and the RDP session uses Guacamole's clipboard sidebar. Mention this in the assignment.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Windows Server administration
- Linux desktop application demos (database GUI tools, monitoring clients)
- IDE-based workshops requiring a desktop VS Code
- GUI-based configuration tools without web interfaces

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

## Multi-Connection User-Mapping

A single Guacamole instance can provide access to multiple VMs or multiple sessions on the same VM. Define multiple `<connection>` blocks inside the `<authorize>` element:

```xml
<user-mapping>
  <authorize username="instruqt" password="instruqt">
    <connection name="Windows Server">
      <protocol>rdp</protocol>
      <param name="hostname">windows-vm</param>
      <param name="port">3389</param>
      <param name="username">Administrator</param>
      <param name="password">instruqt</param>
      <param name="ignore-cert">true</param>
      <param name="security">any</param>
      <param name="resize-method">display-update</param>
    </connection>
    <connection name="Linux Desktop">
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
```

Each connection gets its own service tab with a path that targets it:

```yaml
tabs:
  - title: Windows Server
    type: service
    hostname: guacamole
    port: 8080
    path: /#/client/Windows%20Server

  - title: Linux Desktop
    type: service
    hostname: guacamole
    port: 8080
    path: /#/client/Linux%20Desktop
```

## Deep Link Path Format (v1 vs v2)

The `path:` value in the service tab depends on the Guacamole image version:

| Version | Path format | Example |
|---------|-------------|---------|
| v1 (most Instruqt images) | `/#/client/<ConnectionName>` | `/#/client/Desktop` |
| v2 | `/#/client/<base64-encoded-connection-id>` | `/#/client/MQBjAG0AeQBzAHEAbA==` |

For v1, the connection name must match exactly (case-sensitive, URL-encoded if it contains spaces). For v2, the connection ID is a base64-encoded string of the form `<id>\0c\0<provider>` — check the Guacamole v2 docs for the encoding.

Most Instruqt tracks use the `gcr.io/instruqt/guacamole` image which is v1.

## Additional RDP Parameters

Common parameters for fine-tuning the RDP experience:

```xml
<connection name="Desktop">
  <protocol>rdp</protocol>
  <param name="hostname">workstation</param>
  <param name="port">3389</param>
  <param name="username">root</param>
  <param name="password">instruqt</param>

  <!-- Connection handling -->
  <param name="ignore-cert">true</param>
  <param name="security">any</param>

  <!-- Display -->
  <param name="resize-method">display-update</param>
  <param name="enable-font-smoothing">true</param>
  <param name="enable-wallpaper">false</param>
  <param name="enable-theming">true</param>
  <param name="color-depth">24</param>

  <!-- Clipboard and drive -->
  <param name="disable-copy">false</param>
  <param name="disable-paste">false</param>
  <param name="enable-drive">true</param>
  <param name="drive-path">/shared</param>

  <!-- Performance tuning -->
  <param name="enable-full-window-drag">false</param>
  <param name="enable-menu-animations">false</param>
</connection>
```

Key parameters:
- `enable-font-smoothing: true` — makes text legible at lower resolutions
- `enable-wallpaper: false` — reduces bandwidth
- `enable-drive: true` + `drive-path` — maps a host directory as a drive in the RDP session for file transfer
- `color-depth: 24` — good balance of quality vs bandwidth (16 for low bandwidth, 32 for best quality)

## Common Pitfalls

- **RDP not ready**. The Guacamole setup script must poll `nc -z workstation 3389` before completing. If Guacamole starts before xrdp is listening, the connection fails.
- **Password with XML special characters**. If the RDP password contains `<`, `>`, `&`, `"`, or `'`, the user-mapping.xml will be malformed. Use alphanumeric passwords.
- **Guacamole v1 vs v2 path format**. V1 uses `/#/client/<ConnectionName>`. V2 uses base64-encoded connection strings. Check the image version. Most Instruqt images use v1.
- **Desktop environment weight**. Full GNOME/KDE desktops are heavy. XFCE is recommended. Size the VM to at least `n1-standard-4`.
- **Screen resolution**. Use `resize-method: display-update` in the Guacamole config to auto-resize to the browser window.
- **Clipboard sharing**. Copy-paste between the learner's machine and the RDP session uses Guacamole's clipboard sidebar. Mention this in the assignment.
- **Connection name URL encoding**. Spaces in connection names must be URL-encoded in the service tab path (`Desktop%20Admin` not `Desktop Admin`).

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Windows Server administration
- Linux desktop application demos (database GUI tools, monitoring clients)
- IDE-based workshops requiring a desktop VS Code
- GUI-based configuration tools without web interfaces

# KasmVNC

Browser-based GUI application via a KasmVNC Docker container running on a VM. Use for specific GUI applications (Cursor IDE, branded desktops, custom tools) without provisioning a full RDP desktop environment. Lighter weight than Guacamole RDP.

## Requirements

- A VM with at least `n1-standard-4` and `nested_virtualization: true` (Docker required).
- `allow_external_ingress` with `high-ports`.
- Exposed via a service tab pointing at the KasmVNC port.

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

curl -fsSL https://get.docker.com | sh

docker pull kasmweb/desktop:1.16.1

mkdir -p /workspace
chown 1000:1000 /workspace

docker run -d \
  --name desktop \
  --restart unless-stopped \
  -p 8080:6901 \
  -e VNC_PW=instruqt \
  -e LAUNCH_URL=http://localhost:8080 \
  -v /workspace:/workspace \
  --shm-size=2g \
  kasmweb/desktop:1.16.1

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
```

## Application-Specific Images

KasmVNC provides images for common tools:

| Image | Application |
|-------|-------------|
| `kasmweb/desktop:1.16.1` | Full Xfce desktop |
| `kasmweb/chrome:1.16.1` | Chromium browser only |
| `kasmweb/firefox:1.16.1` | Firefox browser only |
| `kasmweb/terminal:1.16.1` | Terminal emulator |

For custom applications, build on a Kasm base image:

```dockerfile
FROM kasmweb/core-ubuntu-jammy:1.16.1
RUN apt-get update && apt-get install -y your-app
ENV LAUNCH_URL=http://localhost:8080
```

## When to Use

- Track needs a specific GUI application running in a container (not a full desktop).
- Lighter alternative to Guacamole RDP when you don't need Windows or a full desktop environment.
- The GUI application has a Docker-compatible image or can be packaged into one.

See `decision-frameworks/gui-access-method.md` for a comparison with Guacamole and noVNC.

## Common Pitfalls

- **Container UID mismatch**. KasmVNC runs as UID 1000. Mounted volumes must be `chown 1000:1000`, or the container cannot write to them.
- **Docker image pull time**. KasmVNC images are 1-2 GB. Pre-pull into a custom Packer image for production tracks.
- **Shared memory**. KasmVNC needs `--shm-size=2g` or more. The default 64 MB causes visual glitches and crashes.
- **Port mapping**. KasmVNC's internal port is 6901. Map it to your desired host port (e.g., `-p 8080:6901`).
- **Chromium --no-sandbox**. If running Chromium inside the container as root, it requires `--no-sandbox`. Some Kasm images handle this; custom images may need a wrapper.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Cursor IDE workshops (custom KasmVNC image with Cursor installed)
- Branded desktop environments for product demos
- Browser-based testing (Chromium or Firefox in an isolated container)
- GUI application tutorials where the app runs in a container

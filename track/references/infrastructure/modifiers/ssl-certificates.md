# SSL Certificates

Instruqt-managed TLS certificate provisioning for VMs. Use when your track serves a web application over HTTPS and needs a valid certificate without runtime certificate generation.

## What It Provides

Setting `provision_ssl_certificate: true` on a VM causes Instruqt to provision a TLS certificate for that VM's public hostname (`<vm>-<sandbox-id>.instruqt.io`). The certificate and private key are available via the GCP instance metadata endpoint inside the VM.

## Config.yml Example

```yaml
version: "3"
virtualmachines:
  - name: webserver
    image: ubuntu-os-cloud/ubuntu-2204-lts
    machine_type: n1-standard-2
    shell: /bin/bash
    allow_external_ingress:
      - https                                     # required for HTTPS access
    provision_ssl_certificate: true
```

### Retrieving the Certificate

In a setup script, fetch the certificate and key from the GCP metadata endpoint:

```bash
#!/bin/bash
METADATA="http://metadata.google.internal/computeMetadata/v1/instance/attributes"
HEADER="Metadata-Flavor: Google"

curl -s -H "$HEADER" "$METADATA/ssl-certificate" > /etc/ssl/certs/instruqt.crt
curl -s -H "$HEADER" "$METADATA/ssl-certificate-key" > /etc/ssl/private/instruqt.key
chmod 600 /etc/ssl/private/instruqt.key
```

Then configure your web server (nginx, Caddy, etc.) to use these files.

## Common Pitfalls

- **Missing `allow_external_ingress: [https]`**. The certificate is provisioned, but HTTPS traffic is blocked if `https` is not in the ingress list. Both settings are required together.
- **Using certbot or Let's Encrypt instead**. Runtime ACME challenges are unreliable in sandboxes -- DNS propagation delays and rate limits cause intermittent failures. Always prefer `provision_ssl_certificate` for Instruqt VMs.
- **Certificate not yet available during early setup**. The certificate is provisioned asynchronously. If your setup script runs before it arrives, add a retry loop on the metadata fetch.
- **Containers don't support this**. `provision_ssl_certificate` is a VM-only feature. For containers needing HTTPS, use a reverse proxy pattern or a virtual browser tab.
- **Forgetting the key permissions**. The private key must be readable only by the web server process. A world-readable key may cause security warnings or server refusal to start.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Web application deployment lab where learners configure nginx with TLS
- API gateway workshop with HTTPS endpoints
- OAuth/OIDC tutorial requiring HTTPS callback URLs
- Service mesh demo with mTLS between services

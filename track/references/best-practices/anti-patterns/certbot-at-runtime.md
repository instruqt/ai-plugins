---
severity: advisory
---
# Certbot at Runtime

Running `certbot certonly` at runtime when `provision_ssl_certificate: true` is available.

## Why It's Bad

- **Rate limits**: Let's Encrypt enforces 5 certificates per registered domain per week. A popular track with concurrent learners will hit this quickly.
- **Silent failures**: When the rate limit is hit, certbot fails silently and the service runs without TLS, confusing learners.
- **Startup delay**: certbot adds 30-60 seconds to setup while it completes the ACME challenge.

## What to Do Instead

Set `provision_ssl_certificate: true` on the VM in config.yml. Instruqt provisions the certificate through GCP instance metadata before the setup script runs. Read the certificate from the metadata endpoint if needed.

## Examples

Bad -- running certbot in a setup script:

```bash
# setup-sandbox
certbot certonly --standalone -d $HOSTNAME.instruqt.io --agree-tos -m admin@example.com
```

Good -- using Instruqt's built-in SSL provisioning:

```yaml
# config.yml
virtualmachines:
  - name: sandbox
    provision_ssl_certificate: true
```

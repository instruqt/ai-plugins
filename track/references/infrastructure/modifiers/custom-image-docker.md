# Custom Docker Container Image

Pre-built container images that bundle tools, libraries, and configuration beyond what the stock Instruqt images provide. Use when the base `gcr.io/instruqt/shell` or `gcr.io/instruqt/cloud-client:2` images lack the tools you need and installing them at runtime is slow or fragile.

## What It Provides

A custom Docker image referenced by the `image:` field under `containers` in `config.yml`. You build the image with a standard Dockerfile, push it to a container registry (Docker Hub, GCR, Artifact Registry), and Instruqt pulls it at sandbox start.

## Config.yml Example

```yaml
version: "3"
containers:
  - name: shell
    image: docker.io/myorg/workshop-tools:1.2    # custom image from Docker Hub
    ports:
      - 8080
    memory: 512
    shell: /bin/bash
```

### Dockerfile Approach (Overview)

```dockerfile
FROM gcr.io/instruqt/cloud-client:2

RUN apt-get update && apt-get install -y \
    jq \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

COPY config/ /etc/workshop/
```

1. Base on an Instruqt image or any public image that fits your needs.
2. Install additional packages, copy config files, set environment defaults.
3. Push to a registry accessible from the Instruqt sandbox network.
4. Reference the full image path in `config.yml`.

## Registry Notes

- **Docker Hub**: Instruqt sandboxes pull through a Docker Hub registry mirror, so Docker Hub images work without authentication.
- **GCR / Artifact Registry**: If the image is in a private GCP registry, the Instruqt sandbox service account needs read access.
- **Tag pinning**: Always use a specific tag (`:1.2`, `:2024-06`) rather than `:latest`. Untagged images cause inconsistent learner experiences when the image updates mid-track-lifecycle.

## Common Pitfalls

- **Large images increase pull time**. Container images are pulled fresh at sandbox start. Keep images small -- use multi-stage builds, clean up caches, and avoid unnecessary layers.
- **Overriding ENTRYPOINT without setting `entrypoint:` in config.yml**. If your Dockerfile sets an ENTRYPOINT that conflicts with Instruqt's shell expectations, the container may fail to start. Set `entrypoint: bash` in `config.yml` to override.
- **Baking secrets into the image**. Never embed API keys, tokens, or credentials in a Dockerfile. Use Instruqt `secrets:` for runtime credentials.
- **Forgetting to expose ports**. Ports must be listed both in the Dockerfile (`EXPOSE`) and in `config.yml` (`ports:`) for tabs and cross-container access to work.
- **Using Alpine when tools expect glibc**. Some binaries (Terraform plugins, compiled Go tools) fail on Alpine's musl libc. Use a Debian-based image when in doubt.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Security scanning workshop needing Trivy, Grype, and Syft pre-installed
- Data engineering lab with custom Python environment and Jupyter dependencies
- API testing tutorial with Postman CLI, Newman, and sample collections baked in
- Infrastructure-as-Code track needing Pulumi, cdktf, and language-specific SDKs

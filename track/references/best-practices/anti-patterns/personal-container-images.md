---
severity: advisory
---
# Personal Container Images

Using container images from personal registries or accounts in track infrastructure configuration.

## Why It's Bad

Images hosted under personal accounts (e.g., `quay.io/johndoe/my-image`, `docker.io/personaluser/tool:latest`) create a fragile dependency:

- The image disappears if the maintainer deletes it or leaves the organization.
- The image may be updated or removed without notice, breaking the track silently.
- There is no organizational control over image contents, versioning, or security scanning.

## What to Do Instead

Use images from organization-controlled registries, the Instruqt image catalog (`gcr.io/instruqt/`), or well-established public images with pinned tags. For custom images, host them under an org-scoped registry with access controls.

## Examples

Bad -- personal registry:

```yaml
containers:
- name: workstation
  image: quay.io/johndoe/python-caddy
```

Good -- org-scoped registry:

```yaml
containers:
- name: workstation
  image: gcr.io/my-org-instruqt/python-caddy:1.2.0
```

Good -- Instruqt-provided image:

```yaml
containers:
- name: cloud-client
  image: gcr.io/instruqt/cloud-client
```

Good -- well-established public image with pinned tag:

```yaml
containers:
- name: database
  image: postgres:15.4
```

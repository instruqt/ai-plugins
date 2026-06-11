# Invalid GCP Role

Using `roles/writer` as a GCP IAM role -- it does not exist.

## Why It's Bad

`roles/writer` is not a valid GCP IAM role. The legacy basic roles are `roles/viewer`, `roles/editor`, and `roles/owner`. Using a nonexistent role silently fails during sandbox provisioning: no error is raised, but the service account lacks the intended permissions. Operations that need write access fail at runtime with opaque permission-denied errors.

## What to Do Instead

Use the correct predefined role for the specific service. For Cloud Storage write access, use `roles/storage.objectCreator` or `roles/storage.objectUser`. Prefer narrow, service-specific roles over broad basic roles.

## Examples

Bad:

```yaml
# config.yml
gcp_projects:
  - id: project
    roles:
      - roles/writer
```

Good:

```yaml
# config.yml
gcp_projects:
  - id: project
    roles:
      - roles/storage.objectUser
```

Or for broader access:

```yaml
    roles:
      - roles/editor
```

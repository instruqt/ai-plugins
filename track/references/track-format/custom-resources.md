# Custom Resources

Terraform modules provisioned as sandbox resources, created ahead of other sandbox components. Used when a track needs infrastructure that goes beyond Instruqt's built-in VM/container/cloud primitives.

## Structure

Custom resources are **not** defined in `config.yml`. Configuration is done entirely through the Instruqt web UI in two steps:

1. **Organization-level import** -- publish the Terraform module to the Terraform Registry, then import it into the Instruqt organization.
2. **Per-track attach** -- attach the imported resource to a specific track and configure its input variables.

## Prerequisites

- The Terraform module must be published to the [Terraform Registry](https://registry.terraform.io/).
- The module is imported at the organization level before it can be used by any track.

## Fields / Conventions

### Input Variables

All Terraform input variables defined by the module are configurable in the UI, with one exception:

| Reserved Name | Description |
|---------------|-------------|
| `sandbox_id` | Reserved by Instruqt. Cannot be used as a module input variable name. |

### Output-to-Environment-Variable Mapping

Terraform outputs are automatically exposed as environment variables in lifecycle scripts. The naming convention is:

```
${RESOURCE_NAME}_${OUTPUT_NAME}
```

Both the resource name and output name are uppercased.

**Example:** A custom resource named `sqldb` with a Terraform output `ENDPOINT` becomes the environment variable `$SQLDB_ENDPOINT`, available in all lifecycle scripts.

## Observable Signal in Codebase

When a track uses custom resources (especially for cloud infrastructure), you may see a cluster of cloud credential secrets (e.g., Azure `ARM_*` variables) with **no** corresponding `azure_subscriptions:` block in `config.yml`. This is the hallmark of the custom-resource pattern -- the cloud infrastructure is provisioned via Terraform rather than Instruqt's native cloud integration.

## Examples

### Environment Variable Access

If the organization imports a custom resource named `sqldb` that outputs `ENDPOINT` and `PASSWORD`:

```bash
#!/bin/bash
set -euxo pipefail

echo "Connecting to database at $SQLDB_ENDPOINT"
mysql -h "$SQLDB_ENDPOINT" -p"$SQLDB_PASSWORD" -e "SHOW DATABASES;"
```

### Typical Use Cases

- Shared databases provisioned before the sandbox starts
- Cloud networking infrastructure (VPCs, subnets, peering)
- Pre-provisioned Kubernetes clusters via Terraform
- Third-party SaaS resource setup

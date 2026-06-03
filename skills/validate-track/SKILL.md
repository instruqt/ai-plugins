# Validate Track

Enforce platform limits, conventions, and correctness checks on a generated track.

## Progress reporting

Do not create your own top-level task list. The invoking command owns the user-facing task list. If this skill represents a single user-visible step in that list, your work maps to one entry. If you need finer-grained progress, update the existing task list rather than starting a new one.

## Validation Checks

### Platform Limits

| Limit | Threshold | How to Check |
|-------|-----------|--------------|
| Containers + websites | Max 15 total | Count `containers` + `virtualbrowsers` entries in config.yml |
| VMs per cloud provider | Max 4 | Count `virtualmachines` entries in config.yml |
| Script timeout (check) | 60 seconds | Scan check scripts for `sleep`, polling loops, long-running commands |
| Script size | < 5 MB per script | Check file sizes of all scripts |
| Service tab ↔ port alignment | Every service tab hostname:port must match a config.yml resource | Cross-reference assignment.md tab definitions with config.yml ports |
| Website tab URLs | Must be HTTPS | Scan all `type: website` tabs for non-HTTPS URLs |

### YAML Validation

- `track.yml` must be valid YAML with required fields: `slug`, `title`, `teaser`, `description`, `type`
- `config.yml` must be valid YAML (if present — not needed with sandbox presets)
- `assignment.md` frontmatter must be valid YAML between `---` delimiters
- All challenge slugs must match their directory names

### Script Checks

- All scripts must have `#!/bin/bash` shebang
- Setup/solve/cleanup scripts must have `set -euxo pipefail`
- Check scripts must NOT have `set -euo pipefail` or `set -e`
- Run `shellcheck` on all scripts (if available)
- Scripts must be executable (`chmod +x`)

### Structural Checks

- Challenge directories follow `NN-slug` numbering convention
- Every challenge has an `assignment.md`
- Tab references in assignment body match actual tab indices
- Variable interpolation syntax uses `[[ Instruqt-Var ]]` not `{{ }}`

### Error References

Load error diagnosis docs from `references/errors/` when validation fails:
- `references/errors/script-errors.md` — script execution failures
- `references/errors/config-errors.md` — config.yml issues
- `references/errors/push-validation-errors.md` — `instruqt track push` failures

## Workflow

1. Read config.yml and check platform limits
2. Validate YAML syntax across all files
3. Check all scripts for convention compliance
4. Verify structural integrity (directories, files, naming)
5. Run shellcheck on all bash scripts
6. Report findings grouped by severity (blocking → warning → suggestion)

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

Shebang and set flags must match the host image (see `write-scripts` → "Shebang and Set Flags"):
- bash host: `#!/bin/bash`; setup/solve/cleanup use `set -euxo pipefail`
- Alpine/POSIX-sh host (and the bootstrap script that installs bash): `#!/bin/sh`; setup/solve/cleanup use `set -eu`

Plus, regardless of shell:
- Check scripts must NOT have `set -e` or `set -o pipefail` — they must run every assertion
- Run `shellcheck` on all scripts (if available)
- Scripts must be executable (`chmod +x`)

### Prerequisite Verification

Cross-check each challenge's Prerequisites manifest against its setup scripts. The manifest lives in the challenge plan (`${TRACK_OUTPUT_DIR}/.instruqt/<NN-slug>/plan.md`, "Prerequisites" section); track-level capabilities live in the track plan (`${TRACK_OUTPUT_DIR}/.instruqt/plan.md`, "Track-Level Prerequisites").

- Every manifest row marked "Provided by: this challenge" (or the track plan rows, for `track_scripts/setup-<host>`) must have a matching assertion in the corresponding setup script — the Verify command, or an equivalent functional check, appears in the script's verification tail.
- Flag manifest rows whose Verify command only tests existence (`command -v`, `which`, `[ -f ... ]`) when the capability implies function (auth, a reachable backend, an importable package) — these pass at setup but fail at runtime.
- Flag setup scripts that install a tool/runtime/package or start a service but have no verification tail at all.
- Do not require re-verification of capabilities marked "Provided by: track setup" or a prior challenge slug in per-challenge setup — those are verified where they are provided.

If a challenge plan has no Prerequisites section, note it as a warning (the plan predates this convention) rather than blocking.

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

## CLI Validation

After internal checks, run the Instruqt CLI validator if available:

```bash
# Check if instruqt CLI is available
if command -v instruqt &>/dev/null; then
  instruqt track validate --path "${TRACK_OUTPUT_DIR}"
fi
```

This catches issues the internal checks cannot: server-side field validation, slug uniqueness within the org, and format requirements that may change between platform versions.

### Shellcheck

Run shellcheck on all bash lifecycle scripts if the tool is available:

```bash
if command -v shellcheck &>/dev/null; then
  find "${TRACK_OUTPUT_DIR}" -name 'setup-*' -o -name 'check-*' -o -name 'solve-*' -o -name 'cleanup-*' \
    | xargs shellcheck --severity=warning
fi
```

If shellcheck is not installed, note it as a warning but do not block generation.

## Workflow

1. Read config.yml and check platform limits
2. Validate YAML syntax across all files
3. Check all scripts for convention compliance
4. Cross-check Prerequisites manifests against setup verification tails
5. Verify structural integrity (directories, files, naming)
6. Run shellcheck on all bash scripts (if available)
7. Run `instruqt track validate` (if CLI available)
8. Report findings grouped by severity (blocking → warning → suggestion)

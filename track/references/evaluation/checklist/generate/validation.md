# Validation

Binary checks for tool-based validation of generated track artifacts. These run automated tools and check for clean output.

## Shellcheck passes on all scripts

Run `shellcheck` on every shell script in the track.

- **1 (pass):** All scripts produce zero shellcheck warnings.
- **0 (fail):** One or more scripts produce shellcheck warnings.

## YAML files valid

Verify that all YAML files in the track (config.yml, track.yml) parse without errors.

- **1 (pass):** All YAML files parse cleanly with no syntax errors.
- **0 (fail):** One or more YAML files have syntax errors.

## Script permissions are executable

Verify that all shell scripts have the executable bit set.

- **1 (pass):** All script files have executable permissions.
- **0 (fail):** One or more script files are missing the executable bit.

## Quota limits respected

Verify that config.yml resource definitions stay within Instruqt platform quota limits for machine types, disk sizes, and cloud account resources.

- **1 (pass):** All resource definitions are within platform quota limits.
- **0 (fail):** One or more resources exceed platform quota limits.

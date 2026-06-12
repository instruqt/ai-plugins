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

## No exit 0 masking in check scripts

Verify that check scripts do not end with an unconditional `exit 0` that masks validation failures.

- **1 (pass):** No check script ends with a bare `exit 0` after its validation logic.
- **0 (fail):** One or more check scripts have `exit 0` at the end, masking failures.

## No || true on validation commands in check scripts

Verify that check scripts do not use `|| true` on validation commands, which prevents checks from failing.

- **1 (pass):** No `|| true` on validation/assertion commands in check scripts.
- **0 (fail):** One or more check scripts use `|| true` on validation commands.

## No chmod 777 in scripts

Verify that scripts do not use `chmod 777` on files or directories.

- **1 (pass):** No scripts use `chmod 777`.
- **0 (fail):** One or more scripts use `chmod 777`.

## No hardcoded credentials in scripts

Verify that scripts do not contain hardcoded passwords, tokens, or API keys -- including in git clone URLs, docker run commands, and dotfile exports.

- **1 (pass):** No scripts contain hardcoded credentials.
- **0 (fail):** One or more scripts contain hardcoded credentials.

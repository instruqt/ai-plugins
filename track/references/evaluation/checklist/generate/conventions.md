# Conventions

Binary checks for generated track file structure conventions. These verify that the generated track follows the standard patterns for file organization, naming, and script structure.

## Script shebangs correct

Verify that all shell scripts have a correct shebang line (e.g., `#!/bin/bash`) as the first line.

- **1 (pass):** All scripts have a valid shebang line.
- **0 (fail):** One or more scripts are missing a shebang or have an invalid shebang.

## Set flags correct per script type

Verify that set flags match the script type: check scripts do NOT use `set -e`, `set -o pipefail`, or `set -euo pipefail`. Setup, solve, and cleanup scripts use `set -euxo pipefail`.

- **1 (pass):** All scripts use the correct set flags for their type.
- **0 (fail):** One or more scripts use incorrect set flags (e.g., a check script with `set -euo pipefail`, or a setup script missing `set -euxo pipefail`).

## Challenge directory naming follows NN-slug pattern

Verify that challenge directories follow the `NN-descriptive-slug` naming pattern (e.g., `01-install-the-cli`, `02-configure-the-cluster`).

- **1 (pass):** All challenge directories match the `NN-slug` pattern with zero-padded numbers and descriptive slugs.
- **0 (fail):** One or more challenge directories use a different naming pattern.

## Script files in correct locations

Verify that scripts are placed in the correct challenge directories: each challenge's check, solve, setup, and cleanup scripts are inside that challenge's directory, not in a shared location.

- **1 (pass):** All scripts are in their respective challenge directories.
- **0 (fail):** One or more scripts are in the wrong directory or in a shared/root location.

## Setup retry loops have bounded iteration counts

Verify that polling/retry loops in setup scripts have a maximum attempt count and do not loop indefinitely.

- **1 (pass):** All retry/polling loops have a finite iteration count or maximum attempt guard.
- **0 (fail):** One or more retry loops use unbounded `until`/`while` loops with no maximum attempts.

## Solve scripts use set -euxo pipefail

Verify that solve scripts include `set -euxo pipefail` to fail fast on errors.

- **1 (pass):** All solve scripts include `set -euxo pipefail`.
- **0 (fail):** One or more solve scripts are missing `set -euxo pipefail`.

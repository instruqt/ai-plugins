# Script Quality

Evaluates the quality and correctness of generated check, solve, setup, and cleanup scripts. Check scripts must NOT use `set -euo pipefail` or `set -e` -- they test for failure conditions and need to handle non-zero exit codes gracefully. Setup, solve, and cleanup scripts use `set -euxo pipefail` to fail fast on errors.

## Check script patterns

Evaluate whether check scripts follow best practices: one check assertion per script, no pipefail, suppressed stderr, explicit if/else control flow, explicit exit codes, and descriptive fail-message output.

Score 4: All best practices followed across check scripts. Each validates one assertion without pipefail, with suppressed stderr, explicit exit codes, and clear fail-message output on failure.
Score 5: All best practices followed consistently across the entire track. Patterns are uniform and immediately recognizable. Fail-messages reference specific assignment steps to guide the learner.
Score 3: Each script checks one assertion with proper headers, but some miss error suppression or have unclear control flow.
Score 2: Most scripts check one thing, but some use `set -e` or `set -euo pipefail` (causing aborts on expected failures) or lack stderr suppression.
Score 1: Scripts validate multiple assertions, have no headers, and output raw stderr to learners.

## Fail-message quality

Evaluate whether check script failure messages help the learner understand what went wrong and how to fix it.

Score 4: Fail-messages clearly state what was expected, what was found, and which assignment step to revisit. Messages are actionable -- the learner knows what to try next.
Score 5: Fail-messages are precise and pedagogical. They reference the specific assignment step, describe the expected state, hint at common mistakes without giving the answer, and guide the learner toward self-correction.
Score 3: Fail-messages identify the problem area but are too vague to be actionable (e.g., "the configuration is incorrect").
Score 2: Fail-messages exist but are generic (e.g., "check failed") or echo raw command output without interpretation.
Score 1: No fail-messages. Check scripts exit with non-zero codes but provide no guidance to the learner.

## Solve script completeness

Evaluate whether solve scripts complete ALL check assertions in order with realistic values.

Score 4: All check assertions are satisfied by the solve script in the correct order with realistic values.
Score 5: All assertions completed with realistic values, scripts are idempotent, and handle partial completion gracefully.
Score 3: All assertions are addressed, but some values are unrealistic or would not pass in a real scenario.
Score 2: Solve scripts exist but are incomplete or use placeholder values.
Score 1: Solve scripts missing or only cover a subset of check assertions with placeholder values.

## Solve script idempotency

Evaluate whether solve scripts are safe to run multiple times by checking state before acting.

Score 4: Scripts consistently check state before acting with no errors on re-run.
Score 5: Every operation is guarded, re-runs are silent, and all intermediate states are handled correctly.
Score 3: Most operations check state first, but a few would produce warnings on re-run.
Score 2: Some operations are guarded, but others blindly overwrite or duplicate.
Score 1: Scripts fail on second run due to duplicate operations or conflicts.

## Setup script safety

Evaluate whether setup scripts are idempotent, avoid pre-configuring learning objectives, handle bootstrap sentinel waits for dependent services, and verify that the prerequisites the challenge depends on are actually functional (not merely installed).

Score 4: Idempotent, cleanly separate infrastructure setup from learning objectives. Bootstrap sentinel waits ensure dependent services are ready before the challenge starts. A verification tail asserts each capability the script provides is functional — authenticated CLIs, importable packages under the interpreter the learner uses, services accepting connections — failing loudly with a located message if any is not.
Score 5: Fully idempotent, strict separation between setup and learning. Bootstrap waits have explicit timeouts. Hot-start edge cases are handled -- setup adapts if prior challenge state exists. Verification tail covers every capability with functional (not existence-only) checks, uses bounded readiness polling for capabilities that take time to become true, and maps cleanly to the challenge plan's Prerequisites manifest.
Score 3: Idempotent, mostly avoid pre-configuring learning objectives, but missing service readiness waits that could cause race conditions, or the verification tail checks only existence (`command -v`) where function is required (auth, reachable backend, importable package).
Score 2: Partially idempotent with some overlap between setup and learning objectives, or no verification tail — tools/runtimes/packages are installed but never confirmed usable.
Score 1: Scripts fail on re-run, pre-configure what learners should do themselves, and have no service readiness checks or prerequisite verification.

## Shell best practices

Evaluate whether scripts follow shell scripting best practices: correct shebangs, appropriate set flags per script type (no pipefail for check scripts, `set -euxo pipefail` for setup/solve/cleanup), quoted variables, error handling, and consistent style.

Score 4: Consistent headers with correct set flags per script type. Quoted variables, proper error handling, uniform style throughout.
Score 5: Correct headers for shell type and script type. All variables quoted, stderr suppressed in check scripts, explicit exit codes, clean and consistent style across all scripts. Check scripts have no unconditional `exit 0` masking. Setup retry loops have finite iteration counts. Tool installations are version-pinned with checksum verification for direct downloads. Long-running services use systemd unit files instead of nohup.
Score 3: Proper headers with basic error handling, minor quoting or style issues.
Score 2: Shebangs present but incorrect set flags for the script type. Several unquoted variables.
Score 1: No shebangs, no set flags, unquoted variables, mixed indentation and style.

## Timeout awareness

Evaluate whether scripts account for the 60-second execution timeout. Long-running operations should be avoided or optimized.

Score 4: All scripts complete well within the 60-second timeout. No unnecessary sleeps, no unbounded waits, no operations that could hang.
Score 5: Scripts are optimized for speed. Wait loops have explicit timeout guards. Expensive operations are cached or pre-computed in setup scripts to keep check/solve scripts fast.
Score 3: Scripts generally complete in time, but one script has an operation that could approach the timeout under load.
Score 2: One or more scripts contain operations that could exceed 60 seconds -- large downloads, unguarded wait loops, or expensive computations.
Score 1: Scripts contain unbounded waits, large downloads, or operations that will routinely exceed the timeout.

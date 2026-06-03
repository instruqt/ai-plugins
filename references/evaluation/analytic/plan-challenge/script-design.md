# Script Design

Evaluates the quality of planned check, solve, and setup script specifications. Script design should be specific enough that an implementer can write the scripts without guessing what to validate, what state to produce, or what preconditions to establish.

## Check script specifications

Evaluate whether planned check assertions are specific and aligned with assignment steps.

Score 4: Every assignment step has a corresponding check assertion. Each assertion specifies what to test, what constitutes success, and what the fail-message should communicate to the learner.
Score 5: Check assertions anticipate common mistakes and include fail-messages that reference specific assignment steps. Edge cases are considered -- partial completion, wrong values, correct intent but incorrect syntax.
Score 3: Check assertions have specific tests but miss edge cases or do not cover all assignment steps. Some assertions are vague about success criteria.
Score 2: Check scripts are mentioned but specifications are vague (e.g., "check that the user configured the service"). No specific commands or expected values.
Score 1: No check script design, or described checks contradict what the assignment steps ask the learner to do.

## Solve script specifications

Evaluate whether planned solve scripts are specified well enough to produce check-passing state.

Score 4: Solve scripts are specified to produce the exact state that check assertions expect. Commands are specific, values are realistic, and the order matches the assignment flow.
Score 5: Solve scripts are specified as idempotent operations. Each command checks state before acting. The specification handles partial completion gracefully -- running solve after completing some steps manually does not break remaining checks.
Score 3: Solve scripts address all checks but some commands are underspecified or values are placeholder-like.
Score 2: Solve scripts are mentioned but only cover a subset of checks, or use generic placeholder values.
Score 1: No solve script specifications, or specifications would produce state that fails the planned checks.

## Setup script specifications

Evaluate whether planned setup scripts establish the correct preconditions without pre-configuring learning objectives.

Score 4: Setup scripts are specified to create correct preconditions for the challenge. Infrastructure is provisioned, files are placed, services are started -- but nothing the learner should do themselves is pre-configured.
Score 5: Setup scripts handle hot-start edge cases -- they check whether prior challenge state exists and adapt accordingly. Preconditions are precisely specified with no overlap with learning objectives. Wait conditions for dependent services are explicitly planned.
Score 3: Setup scripts establish the right environment but one precondition is either missing or overlaps with what the learner should configure.
Score 2: Setup scripts are mentioned but underspecified. An implementer would need to guess most preconditions.
Score 1: No setup script design, or the planned setup pre-configures what the learner is supposed to learn.

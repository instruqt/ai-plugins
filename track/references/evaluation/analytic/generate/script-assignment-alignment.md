# Script-Assignment Alignment

Evaluates whether check scripts validate what the assignment actually asks the learner to do. Misalignment between assignment content and check assertions is one of the most common defects in generated tracks -- the assignment tells the learner to do X, but the check script tests for Y.

## Check-assignment correspondence

Evaluate whether every assignment step that asks the learner to perform an action has a corresponding check assertion that validates that specific action.

Score 4: Every assignment step has a corresponding check assertion. The check tests for exactly what the assignment asks, not a proxy or side effect.
Score 5: Check assertions map 1:1 to assignment steps with clear fail-messages that reference the specific step. No assignment step is unvalidated, and no check tests something the assignment did not ask for.
Score 3: Check assertions align with the assignment's overall goals but miss one or two specific steps, or test a proxy instead of the actual requested action.
Score 2: Check validates the right general area but specific expectations differ from what the assignment tells the learner to do.
Score 1: Check script tests completely different things from what the assignment asks. A learner following the assignment correctly would fail the check.

## Solve-check consistency

Evaluate whether the solve script produces the exact state that check assertions expect.

Score 4: Solve produces exact check-passing state. Running solve followed by check always succeeds. Values in solve match the values check expects.
Score 5: Solve is documented inline with comments explaining which check each command satisfies. Running solve from any intermediate state (partial completion) produces full check-passing state.
Score 3: Solve passes checks but uses slightly different approaches than the assignment describes -- the end state is correct but the path differs.
Score 2: Solve produces most of the expected state but one or two checks fail after running solve.
Score 1: Solve script does not produce check-passing state. Running solve followed by check fails.

## No phantom checks

Evaluate whether all check assertions correspond to something the assignment explicitly asks the learner to do.

Score 4: Every check assertion maps to an explicit assignment instruction. No checks test for implicit requirements the learner was not told about.
Score 5: Perfect alignment -- checks validate exactly and only what the assignment teaches. The learner is never surprised by a check failure for something not mentioned in the assignment.
Score 3: Most checks correspond to assignment instructions, but one check tests for an implicit requirement that a careful reader might infer but was not stated.
Score 2: Two or more checks test for requirements not mentioned in the assignment. Learners would be surprised by these failures.
Score 1: Multiple checks have no corresponding assignment instruction. The learner cannot pass without guessing what the check expects.

# Script-Assignment Alignment

Evaluates whether what the assignment.md asks the learner to do actually matches what the check script validates, across all challenges in the track.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Check scripts test things the assignment never asked. Learners cannot pass challenges by following the instructions. |
| 2 | Below Standard | Check scripts validate the right general area but specific expectations differ. For example, the assignment says "create a file called config.yaml" but the check script looks for "config.yml". |
| 3 | Adequate | Check scripts align with assignments but edge cases are not handled. For example, the assignment shows `kubectl apply -f file.yaml` but the check only looks for the resource, not whether the file exists. |
| 4 | Good | Every assignment step has a corresponding check assertion. Solve scripts produce exactly the state that passes the check. No mismatches between what is asked and what is validated. |
| 5 | Excellent | Check assertions map 1:1 to assignment steps with clear fail messages referencing the specific step. Solve scripts are documented inline showing which assignment step each block satisfies. |

- Every instruction in the assignment that the learner must act on should have a corresponding assertion in the check script
- The check script should not validate anything the assignment did not ask for
- Filenames, paths, resource names, and values in the assignment must exactly match what the check script expects
- The solve script should be a minimal, correct execution of the assignment instructions -- nothing more, nothing less
- Fail messages in the check script should tell the learner which step they missed or got wrong

## What to Watch For

- Assignments that say "create X" but check scripts look for a differently named or located X
- Check scripts that validate extra state the learner was never told to produce
- Solve scripts that take shortcuts not available to the learner (e.g., using pre-staged files the learner does not have access to)
- Assignments with ambiguous instructions where the check script expects one specific interpretation
- Check scripts with no fail messages, or messages that do not help the learner identify what to fix
- Solve scripts that do not actually pass the check script

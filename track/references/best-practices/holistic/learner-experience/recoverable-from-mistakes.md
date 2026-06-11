# Recoverable From Mistakes

Users will make mistakes. The track should help them recover without getting permanently stuck -- evaluates cross-challenge resilience.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Users who make a mistake are stuck with no recovery path. Check script fail messages are absent or unhelpful. No solve/skip mechanism. A wrong command can break the environment permanently. |
| 2 | Below Standard | Some fail messages exist but most do not explain what went wrong. Recovery from common mistakes requires knowledge the user may not have. Solve scripts are present but incomplete. |
| 3 | Adequate | Fail messages generally explain what is wrong. Solve scripts work as a last resort. However, recovery from some common mistakes (wrong directory, wrong branch) is not addressed in the content. |
| 4 | Good | Fail messages explain the problem and point toward the fix. Solve scripts work correctly. Instructions mention recovery for the most common mistakes. One edge-case recovery scenario is unaddressed. |
| 5 | Excellent | Every challenge has clear, actionable fail messages. Solve scripts provide a reliable escape hatch. Instructions proactively address common mistakes and how to recover from them. The environment is resilient -- wrong commands do not permanently break the track. |

**Instruction-level prevention:**
- Instructions should be unambiguous about where to run commands, which files to edit, and what values to use
- Ambiguous phrasing like "create a file" without specifying the path or directory invites mistakes
- When multiple similar actions are possible (e.g., several branches exist), instructions should be explicit about which one to act on
- Warnings or callouts should appear before steps where a wrong action is hard to undo

**Challenge-level recovery:**
- Check script fail messages explain what's wrong and point toward the fix
- Users can retry challenges without being stuck
- Solve scripts (skip) available as a last resort

**Environment-level recovery:**
- If a user runs a wrong command, can they undo it and try again?
- Setup scripts should be robust enough that the environment doesn't break easily

**Common recovery scenarios to consider:**
- User creates a file in the wrong directory
- User commits to the wrong branch
- User deletes a file they need
- User types a command incorrectly
- User follows an ambiguous instruction and ends up in the wrong state

## What to Watch For

- Instructions that are ambiguous about where or how to perform an action
- Steps where a wrong action is hard to undo but no warning is provided
- Challenges with no failure recovery path (user gets stuck permanently)
- No way to skip past a blocking challenge
- Punishing or unhelpful error messages

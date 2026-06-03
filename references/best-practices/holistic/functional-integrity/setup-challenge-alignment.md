# Setup-Challenge Alignment

When a challenge begins, the environment must already be in the correct state for the user to start working -- setup, check, and solve scripts must be consistent across challenges.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Multiple challenges begin before the environment is ready. Setup scripts have not run, required files are missing, or services are not started. Users are blocked immediately. |
| 2 | Below Standard | Most challenges have the correct preconditions, but one or two challenges begin before their required state is established. Users encounter errors they cannot resolve. |
| 3 | Adequate | All challenges have the correct preconditions from setup scripts. However, one challenge depends on state that is only reliably present if the user completed a prior challenge in the expected way (fragile dependency). |
| 4 | Good | Every challenge's preconditions are met by setup scripts or prior challenge completion. Environment state is predictable. One edge case could be more robust. |
| 5 | Excellent | Every challenge has verified preconditions. Setup scripts establish all required state. The environment is robust against unexpected user actions between challenges. State dependencies are explicit and documented in the YAML and scripts. |

**For each challenge, verify:**
- Are prerequisite files/directories present?
- Are required services running?
- Is the git state correct (right branch, right repo)?
- Has the setup script run before the user sees the assignment?

**Setup-check-solve consistency:**
- The setup script for challenge N+1 should assume only the state that the check script for challenge N validates (or that the solve script for challenge N produces)
- Solve scripts must leave the environment in exactly the state the check script expects
- Check scripts should not validate state that the setup script was responsible for creating

**Example:**
- Challenge 5 requires a Kubernetes namespace to exist -- the setup script creates it before the challenge starts
- A merge conflict challenge requires conflicting branches -- setup or earlier challenges create this state

## What to Watch For

- Challenges that require environment state not yet created
- Challenge N completion breaking challenge N+1's preconditions
- Solve scripts that leave the environment in a state that blocks later challenges
- Setup scripts that silently fail without error handling
- Check scripts that validate setup-created state rather than user actions

# End-to-End Task Flow

Walk through the track mentally from start to finish to verify every challenge can be completed in sequence without blockers.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | The track cannot be completed end-to-end. Multiple challenges block progress due to missing state, broken scripts, or incorrect environment configuration. |
| 2 | Below Standard | The track can be completed but requires workarounds. One or two challenges leave the environment in a state that partially breaks subsequent challenges. Solve scripts may not produce correct state. |
| 3 | Adequate | The track can be completed from start to finish on the happy path. One edge case exists where a user action could leave the environment in a problematic state for a later challenge. Solve scripts work. |
| 4 | Good | The track flows cleanly from start to finish. Each challenge leaves correct state for the next. Solve scripts produce the same state as manual completion. One minor flow issue under unusual conditions. |
| 5 | Excellent | The track has been walked through end-to-end and works flawlessly. Every challenge transitions cleanly to the next. Solve scripts are verified to produce identical state to manual completion. The track is robust against common user deviations. |

Walk through the track mentally:
1. Sandbox starts, setup scripts run for the first challenge
2. User lands on the first challenge with the correct environment
3. Each challenge can be completed with the available environment
4. Completing challenge N leaves the environment in the right state for challenge N+1
5. Every challenge can be solved (solve scripts work and pass the check script)
6. The track can be completed from start to finish without blockers

## What to Watch For

- Challenge N completion breaking challenge N+1's preconditions
- Solve scripts that leave the environment in a state that blocks later challenges
- Circular dependencies between resources

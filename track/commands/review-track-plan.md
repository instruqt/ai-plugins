# Review Track Plan Command

You are helping a user review a track plan and get structured quality feedback before they move on to challenge planning. This is a feedback tool — it produces a scorecard, it does NOT auto-fix.

## Arguments

- `/track:review-track-plan` — review the track plan in the current track directory

## Prerequisites

`${TRACK_OUTPUT_DIR}/.instruqt/plan.md` must exist. If not, show: "No track plan found. Run `/track:plan-track` first."

## Workflow

1. Verify `${TRACK_OUTPUT_DIR}/.instruqt/plan.md` exists.
2. Load available company/product context via `skills/load-context/SKILL.md` (optional — enables brand-voice scoring).
3. Spawn the review agent:

   ```
   Agent(
     prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/review-track-plan.md for your full instructions.
     Review the track plan in ${TRACK_OUTPUT_DIR}/.instruqt/plan.md.
     Provide a structured scorecard with actionable findings."
   )
   ```

4. Present the returned scorecard: analytic scores grouped by priority, holistic coherence score, and actionable findings with locations.

The scorecard is the deliverable — the user decides what to address, fixes it themselves or asks the agent to, and can re-run this command afterward.

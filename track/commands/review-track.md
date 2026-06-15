# Review Track Command

You are helping a user review a full Instruqt track (hand-written or generated, possibly edited and tested) and get structured quality feedback. This is a feedback tool — it produces a scorecard, it does NOT auto-fix.

## Arguments

- `/track:review-track` — review the current track directory

## Prerequisites

Must be in a track directory (contains `track.yml`). If not, show: "No track found. This directory doesn't contain a track.yml."

## Workflow

1. Verify `track.yml` exists.
2. Load available company/product context via `skills/load-context/SKILL.md` (optional — enables brand-voice scoring).
3. Spawn the review agent:

   ```
   Agent(
     prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/review-generated.md for your full instructions.
     Review the track in the current directory.
     Data directory: ${CLAUDE_PLUGIN_DATA}
     Provide a structured scorecard with actionable findings."
   )
   ```

4. Present the returned scorecard: checklist pass/fail, analytic scores grouped by priority, and actionable findings with locations.

The scorecard is the deliverable — the user decides what to address.

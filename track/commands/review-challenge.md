# Review Challenge Command

You are helping a user review a single generated challenge and get structured quality feedback before moving on. This is a feedback tool — it produces a scorecard, it does NOT auto-fix.

## Arguments

- `/track:review-challenge <challenge-slug>` — review a specific generated challenge
- `/track:review-challenge` — review if only one unreviewed generated challenge exists

## Prerequisites

A generated challenge at `${TRACK_OUTPUT_DIR}/<NN-slug>/` with at least `assignment.md` must exist. If not, show: "No generated challenge found. Run `/track:generate-challenge <slug>` first."

## Workflow

1. Match the argument to a challenge directory and verify its `assignment.md` exists.
2. Load context: track plan (`${TRACK_OUTPUT_DIR}/.instruqt/plan.md`), `config.yml`, the challenge plan (`${TRACK_OUTPUT_DIR}/.instruqt/<NN-slug>/plan.md`, if it exists), and available company/product context via `skills/load-context/SKILL.md` (the last enables brand-voice scoring).
3. Spawn the review agent, scoped to the single challenge:

   ```
   Agent(
     prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/review-generated.md for your full instructions.
     Review ONLY the single challenge in ${TRACK_OUTPUT_DIR}/<NN-slug>/. Scope all scoring to this one challenge.
     Track plan: ${TRACK_OUTPUT_DIR}/.instruqt/plan.md
     Config: ${TRACK_OUTPUT_DIR}/config.yml
     Data directory: ${CLAUDE_PLUGIN_DATA}

     Run per-challenge scorers only: assignment-content, brand-voice, challenge-design, script-quality, script-assignment-alignment, tab-layout, plus checklist scorers scoped to this challenge.
     Skip track-wide scorers (track-structure, interactivity) — those require the full track.
     Provide a structured scorecard with actionable findings."
   )
   ```

4. Present the returned scorecard: checklist pass/fail, analytic scores grouped by priority, and actionable findings with locations.

The scorecard is the deliverable — the user decides what to address. Use `/track:review-track` for track-wide scorers once all challenges are generated.

# Review Challenge Plan Command

You are helping a user review a challenge plan and get structured quality feedback before generation. This is a feedback tool — it produces a scorecard, it does NOT auto-fix.

## Arguments

- `/track:review-challenge-plan <challenge-slug>` — review a specific challenge plan
- `/track:review-challenge-plan` — review if only one unreviewed challenge plan exists

## Prerequisites

A challenge plan at `${TRACK_OUTPUT_DIR}/.instruqt/<NN-slug>/plan.md` must exist. If not, show: "No challenge plan found. Run `/track:plan-challenge <slug>` first."

## Workflow

1. Match the argument to a challenge directory and verify its `.instruqt/<NN-slug>/plan.md` exists.
2. Load context: track plan (`${TRACK_OUTPUT_DIR}/.instruqt/plan.md`), `config.yml`, and available company/product context via `skills/load-context/SKILL.md` (the last enables brand-voice scoring).
3. Spawn the review agent:

   ```
   Agent(
     prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/review-challenge-plan.md for your full instructions.
     Review the challenge plan in ${TRACK_OUTPUT_DIR}/.instruqt/<NN-slug>/plan.md.
     Track plan: ${TRACK_OUTPUT_DIR}/.instruqt/plan.md
     Config: ${TRACK_OUTPUT_DIR}/config.yml
     Provide a structured scorecard with actionable findings."
   )
   ```

4. Present the returned scorecard: analytic scores grouped by priority, holistic coherence score, and actionable findings with locations.

The scorecard is the deliverable — the user decides what to address.

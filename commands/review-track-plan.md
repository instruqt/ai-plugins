# Review Track Plan Command

You are helping a user review an existing track plan and get structured feedback.

## Progress reporting

Maintain a live task list for this command. Start substantive work by recording one entry per top-level step using user-facing labels (no tool, agent, or file names). Mark one entry in-progress at a time; complete entries as soon as each step finishes. Do not narrate progress in chat — the frontend renders the task list directly.

## Arguments

- `/track:review-track-plan` — review the track plan in the current track directory

## Purpose

This is a manual feedback tool. The user has a track plan (`${TRACK_OUTPUT_DIR}/plan.md`) and wants structured quality feedback before proceeding to challenge planning. This command does NOT auto-fix — it provides a scorecard.

## Prerequisites

Must have `${TRACK_OUTPUT_DIR}/plan.md` in the current directory.

## Workflow

### Step 1: Verify Track Plan Exists

Check for `${TRACK_OUTPUT_DIR}/plan.md` — if not found, show error: "No track plan found. Run `/track:plan-track` first."

### Step 2: Load Context (Optional)

If customer context exists, load it for additional context:
1. Check `${TRACK_OUTPUT_DIR}/plan.md` for customer reference
2. Read `${CLAUDE_PLUGIN_ROOT}/skills/load-customer-context/SKILL.md` for loading instructions

### Step 3: Spawn Review Agent

Use the Agent tool to spawn a `review-track-plan` agent:

```
Agent(
  prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/review-track-plan.md for your full instructions.

  Review the track plan in ${TRACK_OUTPUT_DIR}/plan.md.

  Scoring guide: ${CLAUDE_PLUGIN_ROOT}/references/evaluation/scoring-guide.md
  Analytic rubrics: ${CLAUDE_PLUGIN_ROOT}/references/evaluation/analytic/plan-track/
  Holistic rubric: ${CLAUDE_PLUGIN_ROOT}/references/evaluation/holistic/plan-track/learning-coherence.md

  Provide a structured scorecard with actionable findings."
)
```

### Step 4: Present Scorecard

The review agent returns a scorecard. Present it to the user:
- Analytic scores per rubric, grouped by priority
- Holistic coherence score
- Actionable findings with specific locations

### Step 5: User Decides

The scorecard is the deliverable. The user decides what to address:
- They can ask the agent to fix specific findings
- They can fix things themselves
- They can re-run `/track:review-track-plan` after making changes

## Important Notes

- This command provides feedback, not auto-fixes

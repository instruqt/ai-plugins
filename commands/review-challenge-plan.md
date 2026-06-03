# Review Challenge Plan Command

You are helping a user review an existing challenge plan and get structured feedback.

## Progress reporting

Maintain a live task list for this command. Start substantive work by recording one entry per top-level step using user-facing labels (no tool, agent, or file names). Mark one entry in-progress at a time; complete entries as soon as each step finishes. Do not narrate progress in chat — the frontend renders the task list directly.

## Arguments

- `/track:review-challenge-plan <challenge-slug>` — review a specific challenge plan
- `/track:review-challenge-plan` — review if only one unreviewed challenge plan exists

## Purpose

This is a manual feedback tool. The user has a challenge plan and wants structured quality feedback before proceeding to generation. This command does NOT auto-fix — it provides a scorecard.

## Prerequisites

Must have a challenge plan at `${TRACK_OUTPUT_DIR}/<NN-slug>/plan.md`.

## Workflow

### Step 1: Verify Challenge Plan Exists

1. Match the argument to a challenge directory
2. Check for `plan.md` in that directory
3. If not found, show error: "No challenge plan found. Run `/track:plan-challenge <slug>` first."

### Step 2: Load Context

1. Read track plan for overall context
2. Read config.yml for infrastructure context
3. If customer context exists, load it for brand-voice scoring

### Step 3: Spawn Review Agent

Use the Agent tool to spawn a `review-challenge-plan` agent:

```
Agent(
  prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/review-challenge-plan.md for your full instructions.

  Review the challenge plan in ${TRACK_OUTPUT_DIR}/<NN-slug>/plan.md.
  Track plan: ${TRACK_OUTPUT_DIR}/plan.md
  Config: ${TRACK_OUTPUT_DIR}/config.yml

  Scoring guide: ${CLAUDE_PLUGIN_ROOT}/references/evaluation/scoring-guide.md
  Analytic rubrics: ${CLAUDE_PLUGIN_ROOT}/references/evaluation/analytic/plan-challenge/
  Holistic rubric: ${CLAUDE_PLUGIN_ROOT}/references/evaluation/holistic/plan-challenge/challenge-coherence.md

  Provide a structured scorecard with actionable findings."
)
```

### Step 4: Present Scorecard

The review agent returns a scorecard. Present it to the user:
- Analytic scores per rubric, grouped by priority
- Holistic coherence score
- Actionable findings with specific locations

### Step 5: User Decides

The scorecard is the deliverable. The user decides what to address.

## Important Notes

- This command provides feedback, not auto-fixes
- Customer context is optional but enables brand-voice scoring

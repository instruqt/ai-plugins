# Review Track Command

You are helping a user review an existing Instruqt track and get structured feedback.

## Progress reporting

Maintain a live task list for this command. Start substantive work by recording one entry per top-level step using user-facing labels (no tool, agent, or file names). Mark one entry in-progress at a time; complete entries as soon as each step finishes. Do not narrate progress in chat — the frontend renders the task list directly.

## Arguments

- `/track:review-track` — review the current track directory

## Purpose

This is a manual feedback tool. The user has a track (hand-written or generated, possibly edited), has tested it, and wants structured quality feedback. This command does NOT auto-fix — it provides a scorecard.

## Prerequisites

Must be in a track directory (contains `track.yml`).

## Workflow

### Step 1: Verify Track Directory

Check for `track.yml` — if not found, show error: "No track found. This directory doesn't contain a track.yml."

### Step 2: Load Context (Optional)

If customer context exists, load it — this enables brand-voice scoring:
1. Check `${TRACK_OUTPUT_DIR}/plan.md` for customer reference
2. Read `${CLAUDE_PLUGIN_ROOT}/skills/load-customer-context/SKILL.md` for loading instructions

### Step 3: Spawn Review Agent

Use the Agent tool to spawn a `review-generated` agent:

```
Agent(
  prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/review-generated.md for your full instructions.

  Review the track in the current directory.
  Customer context: <company-slug or 'none'>

  Scoring guide: ${CLAUDE_PLUGIN_ROOT}/references/evaluation/scoring-guide.md
  Checklist rubrics: ${CLAUDE_PLUGIN_ROOT}/references/evaluation/checklist/generate/
  Analytic rubrics: ${CLAUDE_PLUGIN_ROOT}/references/evaluation/analytic/generate/

  Provide a structured scorecard with actionable findings."
)
```

### Step 4: Present Scorecard

The review agent returns a scorecard. Present it to the user:
- Checklist pass/fail results
- Analytic scores per rubric, grouped by priority
- Actionable findings with specific locations

### Step 5: User Decides

The scorecard is the deliverable. The user decides what to address:
- They can ask the agent to fix specific findings
- They can fix things themselves
- They can re-run `/track:review-track` after making changes

## Important Notes

- This command provides feedback, not auto-fixes
- Customer context is optional but enables brand-voice scoring

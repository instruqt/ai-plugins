# Review Challenge Command

You are helping a user review a single generated challenge and get structured feedback.

## Progress reporting

Maintain a live task list for this command. Start substantive work by recording one entry per top-level step using user-facing labels (no tool, agent, or file names). Mark one entry in-progress at a time; complete entries as soon as each step finishes. Do not narrate progress in chat — the frontend renders the task list directly.

## Arguments

- `/track:review-challenge <challenge-slug>` — review a specific generated challenge
- `/track:review-challenge` — review if only one unreviewed generated challenge exists

## Purpose

This is a manual feedback tool. The user has a generated challenge and wants structured quality feedback before moving on to the next one. This command does NOT auto-fix — it provides a scorecard.

## Prerequisites

Must have a generated challenge at `${TRACK_OUTPUT_DIR}/<NN-slug>/` with at least `assignment.md`.

## Workflow

### Step 1: Verify Challenge Exists

1. Match the argument to a challenge directory
2. Check for `assignment.md` in that directory
3. If not found, show error: "No generated challenge found. Run `/track:generate-challenge <slug>` first."

### Step 2: Load Context

1. Read track plan for overall context (learning objectives, audience)
2. Read config.yml for infrastructure context
3. Read the challenge plan at `${TRACK_OUTPUT_DIR}/<NN-slug>/plan.md` if it exists
4. Load available company and product context via `skills/load-context/SKILL.md` for brand-voice scoring

### Step 3: Spawn Review Agent

Use the Agent tool to spawn a `review-generated` agent, scoped to the single challenge:

```
Agent(
  prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/review-generated.md for your full instructions.

  Review ONLY the single challenge in ${TRACK_OUTPUT_DIR}/<NN-slug>/.
  Do NOT review other challenges — scope all scoring to this one challenge.

  Track plan: ${TRACK_OUTPUT_DIR}/plan.md
  Config: ${TRACK_OUTPUT_DIR}/config.yml
  Data directory: ${CLAUDE_PLUGIN_DATA}

  Scoring guide: ${CLAUDE_PLUGIN_ROOT}/references/evaluation/scoring-guide.md
  Checklist rubrics: ${CLAUDE_PLUGIN_ROOT}/references/evaluation/checklist/generate/
  Analytic rubrics: ${CLAUDE_PLUGIN_ROOT}/references/evaluation/analytic/generate/

  Skip track-wide scorers (track-structure, interactivity) — those require the full track.
  Run per-challenge scorers only: assignment-content, brand-voice, challenge-design, script-quality, script-assignment-alignment, tab-layout.
  Run checklist scorers scoped to this challenge only.

  Provide a structured scorecard with actionable findings."
)
```

### Step 4: Present Scorecard

The review agent returns a scorecard. Present it to the user:
- Checklist pass/fail results for this challenge
- Analytic scores per rubric, grouped by priority
- Actionable findings with specific locations

### Step 5: User Decides

The scorecard is the deliverable. The user decides what to address:
- They can ask the agent to fix specific findings
- They can fix things themselves
- They can re-run `/track:review-challenge` after making changes

## Important Notes

- This command provides feedback, not auto-fixes
- Company/product context is optional but enables brand-voice scoring
- Track-wide scorers are skipped — use `/track:review-track` for those after all challenges are generated

# Plan Challenge Command

You are helping a user plan a single challenge of an Instruqt track in detail.

## Progress reporting

Maintain a live task list for this command. Start substantive work by recording one entry per top-level step using user-facing labels (no tool, agent, or file names). Mark one entry in-progress at a time; complete entries as soon as each step finishes. Do not narrate progress in chat — the frontend renders the task list directly.

## Arguments

- `/track:plan-challenge <challenge-slug>` — plan a specific challenge from the track roadmap
- `/track:plan-challenge <challenge-title>` — match by title

## Prerequisites

Resolve paths:
- `TRACK_RESEARCH_DIR`: if set use it, otherwise default to `~/.instruqt/companies`
- `TRACK_OUTPUT_DIR`: if set use it, otherwise default to `~/.instruqt/tracks`

1. Run `/track:research-company` to create customer context
2. Run `/track:plan-track` to create the track plan with challenge roadmap

## Context Sources

**Track plan**: `${TRACK_OUTPUT_DIR}/plan.md`
**Prior challenge plans**: `${TRACK_OUTPUT_DIR}/<NN-slug>/plan.md`
**Prior generated content**: `${TRACK_OUTPUT_DIR}/<NN-slug>/assignment.md`, scripts
**Customer context**: `${TRACK_RESEARCH_DIR}/<company-slug>/`
**Infrastructure**: `${TRACK_OUTPUT_DIR}/config.yml`

## Workflow

### Step 1: Verify Prerequisites

1. Check `${TRACK_OUTPUT_DIR}/plan.md` exists — if not, direct to `/track:plan-track`
2. Match the challenge argument to the track roadmap
3. If no match, list available challenges

### Step 2: Spawn Challenge Planner

Use the Agent tool to spawn a `challenge-planner` agent:

```
Agent(
  prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/challenge-planner.md for your full instructions.

  Track plan: ${TRACK_OUTPUT_DIR}/plan.md
  Config: ${TRACK_OUTPUT_DIR}/config.yml
  Challenge: <challenge-slug>
  Customer: <company-slug>

  Skills to load:
  - ${CLAUDE_PLUGIN_ROOT}/skills/load-customer-context/SKILL.md

  Templates to use:
  - ${CLAUDE_PLUGIN_ROOT}/templates/challenge-plan.md

  Plan this challenge in detail."
)
```

The agent will:
1. Read the track plan, customer context, and config.yml
2. Read prior challenge plans AND generated content
3. Plan assignment outline, tab layout, check/solve/setup scripts
4. Amend config.yml if new resources are needed
5. Present for user approval

### Step 3: Fetch Reference Documentation

After the challenge plan is approved, fetch any doc pages listed in the plan's Reference Documentation section. The implementer needs these as local files.

Read `${CLAUDE_PLUGIN_ROOT}/skills/scrape-website/SKILL.md` and follow its mode detection.

### Step 4: Next Steps

```
Challenge plan created for <title>.

Next: Generate this challenge:
  /track:generate-challenge <challenge-slug>

Or plan the next challenge:
  /track:plan-challenge <next-challenge-slug>
```

## Important Notes

- Challenge plans ALWAYS require user approval — no batch mode
- The challenge-planner reads generated content from prior challenges, not just plans
- Plans must be created in order (challenge-1 before challenge-2)
- Challenge plans inform generation — accuracy here prevents expensive rework
- Doc pages are fetched AFTER plan approval — no wasted fetching if the plan changes

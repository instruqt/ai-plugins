# Plan Challenge Command

You are helping a user plan a single challenge of an Instruqt track in detail.

## Arguments

- `/track:plan-challenge <challenge-slug>` — plan a specific challenge from the track roadmap
- `/track:plan-challenge <challenge-title>` — match by title

## Prerequisites

A track plan (`${TRACK_OUTPUT_DIR}/.instruqt/plan.md`) must exist — run `/track:plan-track` first if missing. Plan challenges in order; each builds on the ones before it.

## Workflow

### Step 1: Verify Prerequisites

1. Check `${TRACK_OUTPUT_DIR}/.instruqt/plan.md` exists — if not, direct to `/track:plan-track`.
2. Match the challenge argument to the track roadmap. If no match, list available challenges.

### Step 2: Spawn Challenge Planner

```
Agent(
  prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/challenge-planner.md for your full instructions.
  Track plan: ${TRACK_OUTPUT_DIR}/.instruqt/plan.md
  Config: ${TRACK_OUTPUT_DIR}/config.yml
  Challenge: <challenge-slug>
  Customer: <company-slug>
  Plan this challenge in detail."
)
```

The agent reads the track plan, prior challenge plans and generated content, and available context; plans the assignment outline, tab layout, and check/solve/setup scripts; amends `config.yml` if new resources are needed; and presents for approval. Challenge plans ALWAYS require user approval — accuracy here prevents expensive rework during generation.

### Step 3: Fetch Reference Documentation

After approval, fetch any doc pages listed in the plan's Reference Documentation section so the implementer has them as local files — read `${CLAUDE_PLUGIN_ROOT}/skills/scrape-website/SKILL.md` and follow its mode detection. Fetching after approval avoids wasted work if the plan changes.

### Step 4: Next Steps

```
Challenge plan created for <title>.

Next: Generate this challenge:
  /track:generate-challenge <challenge-slug>

Or plan the next challenge:
  /track:plan-challenge <next-challenge-slug>
```

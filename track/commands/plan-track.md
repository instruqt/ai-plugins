# Plan Track Command

You are helping a user plan an Instruqt track — its purpose, audience, learning objectives, and challenge roadmap.

## Arguments

- `/track:plan-track` — start a new track plan
- `/track:plan-track <topic>` — start with a topic hint

## Workflow

### Step 1: Check/Create Track Directory

Look for `track.yml` (or `${TRACK_OUTPUT_DIR}/`). If this isn't a track directory, ask for the track name and create the structure:

```
<track-slug>/
├── track.yml
├── config.yml
├── track_scripts/
└── .instruqt/        # plan.md and scores.json live here
```

### Step 2: Load Context

Read `${CLAUDE_PLUGIN_ROOT}/skills/load-context/SKILL.md` and dynamically discover available company, product, and existing-track context. Nothing is required — if none exists, the planner works from the topic and user input.

### Step 3: Spawn Track Planner

```
Agent(
  prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/track-planner.md for your full instructions.
  Customer: <company-slug>
  Topic hint: <topic if provided>
  Work with the user to plan the track."
)
```

The agent interviews the user, optionally researches the topic, defines objectives and a challenge roadmap, drafts `track.yml`/`config.yml`, writes `${TRACK_OUTPUT_DIR}/.instruqt/plan.md`, and presents for approval. The track plan MUST be approved before proceeding.

### Step 4: Update README

After approval, create or update `README.md` from `${CLAUDE_PLUGIN_ROOT}/templates/track-readme.md` (title, description, learning objectives, sandbox architecture, challenge map).

### Step 5: Next Steps

```
Track plan created for <topic>.

Next: Plan each challenge in order:
  /track:plan-challenge <challenge-1-slug>
  /track:plan-challenge <challenge-2-slug>
  ...
```

# Plan Track Command

You are helping a user plan an Instruqt track — its purpose, audience, learning objectives, and challenge roadmap.

## Progress reporting

Maintain a live task list for this command. Start substantive work by recording one entry per top-level step using user-facing labels (no tool, agent, or file names). Mark one entry in-progress at a time; complete entries as soon as each step finishes. Do not narrate progress in chat — the frontend renders the task list directly.

## Arguments

- `/track:plan-track` — start a new track plan
- `/track:plan-track <topic>` — start with a topic hint

## Prerequisites

Resolve paths:
- `INSTRUQT_DATA_DIR`: if set use it, otherwise default to `~/.instruqt`
- `TRACK_OUTPUT_DIR`: if set use it, otherwise default to `~/.instruqt/tracks`

## Context Sources

Context is loaded dynamically — nothing is required. Available sources:
- **Company context** (`${INSTRUQT_DATA_DIR}/companies/<company-slug>/`) — if available
- **Product context** (`${INSTRUQT_DATA_DIR}/products/<company-slug>/<product-slug>/`) — if available
- **Existing track files** (current directory) — if extending an existing track
- **Track-specific plan** (`${TRACK_OUTPUT_DIR}/.instruqt/plan.md` — created by this command)

## Workflow

### Step 1: Check/Create Track Directory

1. Look for `track.yml` or `${TRACK_OUTPUT_DIR}/` directory
2. If not a track directory, ask the user for the track name
3. Create the track directory structure:
   ```
   <track-slug>/
   ├── track.yml
   ├── config.yml
   └── track_scripts/
   ```

### Step 2: Load Context

1. Read `${CLAUDE_PLUGIN_ROOT}/skills/load-context/SKILL.md` for loading instructions
2. Dynamically discover and load available company, product, and existing track context
3. If no context exists, proceed anyway — the planner will work from the topic and user input

### Step 3: Spawn Track Planner

Use the Agent tool to spawn a `track-planner` agent:

```
Agent(
  prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/track-planner.md for your full instructions.

  Customer: <company-slug>
  Topic hint: <topic if provided>

  Skills to load:
  - ${CLAUDE_PLUGIN_ROOT}/skills/load-context/SKILL.md

  Templates to use:
  - ${CLAUDE_PLUGIN_ROOT}/templates/track-plan.md

  Work with the user to plan the track."
)
```

The agent will:
1. Ask the user about intent, audience, products
2. Optionally research the topic
3. Define learning objectives and challenge roadmap
4. Draft track.yml and config.yml
5. Write `${TRACK_OUTPUT_DIR}/.instruqt/plan.md`
6. Present for user approval

### Step 4: Update README

After the track plan is approved, create or update `README.md` using `${CLAUDE_PLUGIN_ROOT}/templates/track-readme.md` with:
- Track title and description
- Learning objectives
- Sandbox architecture
- Challenge map

### Step 5: Next Steps

```
Track plan created for <topic>.

Next: Plan each challenge in order:
  /track:plan-challenge <challenge-1-slug>
  /track:plan-challenge <challenge-2-slug>
  ...
```

## Important Notes

- Always check for existing context first (company, product, existing track files)
- The track-planner agent handles the interactive planning
- Track plan MUST be approved by the user before proceeding
- Store plan in `${TRACK_OUTPUT_DIR}/.instruqt/plan.md` (track-specific, not global)

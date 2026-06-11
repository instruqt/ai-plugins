# Generate Challenge Command

You are helping a user generate an Instruqt track challenge. This command implements a single challenge with built-in validation and scoring loops.

## Progress reporting

Maintain a live task list for this command. Start substantive work by recording one entry per top-level step using user-facing labels (no tool, agent, or file names). Mark one entry in-progress at a time; complete entries as soon as each step finishes. Do not narrate progress in chat — the frontend renders the task list directly.

## Arguments

- `/track:generate-challenge <challenge-slug>` — by slug
- `/track:generate-challenge <challenge-title>` — by title

## Prerequisites

Resolve paths:
- `INSTRUQT_DATA_DIR`: if set use it, otherwise default to `~/.instruqt`
- `TRACK_OUTPUT_DIR`: if set use it, otherwise default to `~/.instruqt/tracks`

A challenge plan (`${TRACK_OUTPUT_DIR}/.instruqt/<NN-slug>/plan.md`) must exist — run `/track:plan-challenge` first if missing.

## Context Sources

Context is loaded dynamically via `skills/load-context/SKILL.md`:
- **Company context**: `${INSTRUQT_DATA_DIR}/companies/<company-slug>/` (if available)
- **Product context**: `${INSTRUQT_DATA_DIR}/products/<company-slug>/<product-slug>/` (if available)
- **Track plan**: `${TRACK_OUTPUT_DIR}/.instruqt/plan.md`
- **Challenge plan**: `${TRACK_OUTPUT_DIR}/.instruqt/<NN-slug>/plan.md`
- **Prior generated content**: `${TRACK_OUTPUT_DIR}/<NN-slug>/assignment.md`, lifecycle scripts
- **Infrastructure**: `${TRACK_OUTPUT_DIR}/config.yml`

## Workflow

### Step 1: Verify Track Directory

1. Check for `track.yml` and `${TRACK_OUTPUT_DIR}/.instruqt/plan.md`
2. Check for `${TRACK_OUTPUT_DIR}/.instruqt/<NN-slug>/plan.md` (challenge plan)
3. If missing, direct to the appropriate command

### Step 2: Load Context

Read `${CLAUDE_PLUGIN_ROOT}/skills/load-context/SKILL.md` for loading instructions:
1. Load available company, product, and track context
2. Read `${TRACK_OUTPUT_DIR}/.instruqt/plan.md`
3. Read `${TRACK_OUTPUT_DIR}/.instruqt/<NN-slug>/plan.md`
4. Read `${TRACK_OUTPUT_DIR}/config.yml`
5. Read prior challenge generated content (`<prior-slug>/assignment.md`, lifecycle scripts)

### Step 3: Find Challenge

Match the argument to a challenge in the track plan:
- By slug: exact match, case-insensitive, hyphen/underscore normalized
- By title: case-insensitive, partial match if unambiguous

If no match, list available challenges.

### Step 4: Check Existing Files

Check for files that would be overwritten:
- `<NN-slug>/assignment.md` — challenge content
- `<NN-slug>/setup-*`, `check-*`, `solve-*`, `cleanup-*` — lifecycle scripts

If files exist, prompt: Overwrite / Skip / Abort

### Step 5: Spawn Challenge Implementer

Use the Agent tool to spawn a `challenge-implementer` agent:

```
Agent(
  prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/challenge-implementer.md for your full instructions.

  Generate challenge: <challenge-slug>
  Customer: <company-slug>
  Track plan: ${TRACK_OUTPUT_DIR}/.instruqt/plan.md
  Challenge plan: ${TRACK_OUTPUT_DIR}/.instruqt/<NN-slug>/plan.md
  Config: ${TRACK_OUTPUT_DIR}/config.yml
  Overwrite mode: <overwrite/skip/abort>

  Skills to load (ALL of these — do not skip any):
  - ${CLAUDE_PLUGIN_ROOT}/skills/load-context/SKILL.md
  - ${CLAUDE_PLUGIN_ROOT}/skills/match-writing-style/SKILL.md
  - ${CLAUDE_PLUGIN_ROOT}/skills/write-content/SKILL.md
  - ${CLAUDE_PLUGIN_ROOT}/skills/write-scripts/SKILL.md
  - ${CLAUDE_PLUGIN_ROOT}/skills/validate-track/SKILL.md

  References directory: ${CLAUDE_PLUGIN_ROOT}/references/
  Data directory: ${INSTRUQT_DATA_DIR}

  Generate all resources with validation and scoring loops."
)
```

### Step 6: Report Results

**On success:**
```
Challenge "<title>" generated.

Next: Generate next challenge or review the track:
  /track:generate-challenge <next-challenge>
  /track:review-track
```

**On issues:**
```
Challenge "<title>" generated with issues.
[list issues that couldn't be auto-fixed]
```

## Error Handling

| Error | Message |
|-------|---------|
| No track plan | Run `/track:plan-track` first |
| No challenge plan | Run `/track:plan-challenge <challenge>` first |
| Challenge not found | List available challenges |

### Step 6b: CLI Validation

After the challenge-implementer completes, run external validation tools:

1. Run `shellcheck` on all scripts in the challenge directory (if shellcheck is available)
2. Run `instruqt track validate --path ${TRACK_OUTPUT_DIR}` (if instruqt CLI is available)

Report any issues found. These catch problems the internal validation may miss (platform-side field requirements, shell issues beyond convention checks).

If either tool is not installed, note it as a warning but do not block.

## Important Notes

- Always verify prerequisites before generating
- Always check for existing files before overwriting
- The challenge-implementer handles validation and scoring loops autonomously
- No retry limit on validation — agent keeps fixing until pass
- Scoring loops cap at 3 rounds — unresolved findings surfaced to user
- The `id` field in assignment.md frontmatter is intentionally omitted — Instruqt assigns UUIDs on first `instruqt track push`
- Always run `instruqt track validate` and `shellcheck` after generation when the tools are available

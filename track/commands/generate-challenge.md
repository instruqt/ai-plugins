# Generate Challenge Command

You are helping a user generate a single Instruqt track challenge, with built-in validation and scoring loops.

## Arguments

- `/track:generate-challenge <challenge-slug>` — by slug
- `/track:generate-challenge <challenge-title>` — by title

## Prerequisites

A challenge plan (`${TRACK_OUTPUT_DIR}/.instruqt/<NN-slug>/plan.md`) must exist — run `/track:plan-challenge` first if missing.

## Workflow

### Step 1: Verify and Find Challenge

1. Check `track.yml`, `${TRACK_OUTPUT_DIR}/.instruqt/plan.md`, and the challenge plan `${TRACK_OUTPUT_DIR}/.instruqt/<NN-slug>/plan.md` exist. If missing, direct to the appropriate command (see Error Handling).
2. Match the argument to a challenge in the track plan — by slug (case-insensitive, hyphen/underscore normalized) or by title (partial match if unambiguous). If no match, list available challenges.

### Step 2: Check Existing Files

Check for files that would be overwritten (`<NN-slug>/assignment.md`, `setup-*`, `check-*`, `solve-*`, `cleanup-*`). If any exist, prompt: Overwrite / Skip / Abort.

### Step 3: Spawn Challenge Implementer

```
Agent(
  prompt="Read ${CLAUDE_PLUGIN_ROOT}/agents/challenge-implementer.md for your full instructions.
  Generate challenge: <challenge-slug>
  Customer: <company-slug>
  Track plan: ${TRACK_OUTPUT_DIR}/.instruqt/plan.md
  Challenge plan: ${TRACK_OUTPUT_DIR}/.instruqt/<NN-slug>/plan.md
  Config: ${TRACK_OUTPUT_DIR}/config.yml
  Overwrite mode: <overwrite/skip/abort>
  References directory: ${CLAUDE_PLUGIN_ROOT}/references/
  Data directory: ${CLAUDE_PLUGIN_DATA}
  Generate all resources with validation and scoring loops."
)
```

The implementer loads its own skills and runs validation and scoring loops autonomously — no retry limit on validation; scoring loops cap at 3 rounds, with unresolved findings surfaced.

### Step 4: CLI Validation

After the implementer completes, run external tools before reporting success — they catch platform-side issues internal validation can miss:

1. `shellcheck` on all scripts in the challenge directory (if available).
2. `instruqt track validate --path ${TRACK_OUTPUT_DIR}` (if the Instruqt CLI is available).

If a tool isn't installed, note it as a warning but don't block.

### Step 5: Report Results

**On success:**
```
Challenge "<title>" generated.

Next: Generate next challenge or review the track:
  /track:generate-challenge <next-challenge>
  /track:review-track
```

**On issues:** report what couldn't be auto-fixed or what CLI validation flagged.

## Error Handling

| Error | Message |
|-------|---------|
| No track plan | Run `/track:plan-track` first |
| No challenge plan | Run `/track:plan-challenge <challenge>` first |
| Challenge not found | List available challenges |

## Important Notes

- The `id` field in `assignment.md` frontmatter is intentionally omitted — Instruqt assigns UUIDs on first `instruqt track push`.

# Generate README Command

You are helping a user generate a maintainer-facing `README.md` for their Instruqt track, from the track plan, `config.yml`, and generated challenges.

## Arguments

- `/track:generate-readme` — generate README.md for the current track

## Prerequisites

At minimum the track must have `track.yml`, `config.yml` (unless using a sandbox preset), and at least one challenge directory with `assignment.md`. The track plan (`${TRACK_OUTPUT_DIR}/.instruqt/plan.md`) is optional but improves quality.

## Workflow

### Step 1: Read Track Context

1. `track.yml` — metadata (title, description, timelimit, pausable, ...).
2. `config.yml` — infrastructure (hosts, cloud accounts, ports).
3. `${TRACK_OUTPUT_DIR}/.instruqt/plan.md` if available — objectives, audience, roadmap.
4. Each challenge's `assignment.md` frontmatter — slug, title, tabs.

### Step 2: Generate README

Fill `${CLAUDE_PLUGIN_ROOT}/templates/track-readme.md` with:

- **Track overview** — from the plan's intent, or derived from the `track.yml` description.
- **Sandbox architecture** — ASCII diagram of hosts, ports, and connections from `config.yml`. This is the most valuable part for onboarding contributors.
- **Hosts table** — hostname, image, machine type, role per container/VM.
- **Tabs table** — title, type, hostname, port per challenge's tabs.
- **Challenge map** — ordered list with slug, title, difficulty, time estimate.
- **Deployment** — `instruqt track push` / validate / test commands.
- **Troubleshooting** — common issues for the track's infrastructure pattern.

### Step 3: Write README

Write `${TRACK_OUTPUT_DIR}/README.md`. If one already exists, prompt: Overwrite / Merge / Abort.

## Error Handling

| Error | Message |
|-------|---------|
| No track.yml | This doesn't look like a track directory. |
| No challenges | Generate challenges first, then generate the README. |

## Important Notes

- The README targets authors and maintainers, not learners.
- Do not include customer-specific secrets (API keys, internal URLs) or the `id` fields from `track.yml`.

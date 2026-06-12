# Generate README Command

You are helping a user generate a README.md for their Instruqt track. This command reads the track plan, config.yml, and generated challenges to produce a comprehensive track README.

## Progress reporting

Maintain a live task list for this command. Start substantive work by recording one entry per top-level step using user-facing labels (no tool, agent, or file names). Mark one entry in-progress at a time; complete entries as soon as each step finishes. Do not narrate progress in chat — the frontend renders the task list directly.

## Arguments

- `/track:generate-readme` — generate README.md for the current track

## Prerequisites

Resolve paths:
- `TRACK_OUTPUT_DIR`: if set use it, otherwise default to the current working directory

At minimum, the track must have:
1. `track.yml`
2. `config.yml` (unless using a sandbox preset)
3. At least one challenge directory with `assignment.md`

A track plan (`plan.md`) is optional but improves README quality.

## Context Sources

**Track config**: `${TRACK_OUTPUT_DIR}/track.yml`
**Infrastructure**: `${TRACK_OUTPUT_DIR}/config.yml`
**Track plan**: `${TRACK_OUTPUT_DIR}/.instruqt/plan.md` (if available)
**Challenges**: `${TRACK_OUTPUT_DIR}/*/assignment.md`

## Workflow

### Step 1: Read Track Context

1. Read `track.yml` for track metadata (title, description, timelimit, pausable, etc.)
2. Read `config.yml` for infrastructure details (hosts, cloud accounts, ports)
3. Read track plan if available (learning objectives, audience, challenge roadmap)
4. Read each challenge's `assignment.md` frontmatter for slug, title, tabs

### Step 2: Load README Template

Read `${CLAUDE_PLUGIN_ROOT}/templates/track-readme.md` for the scaffold structure.

### Step 3: Generate README

Fill in the template with information from the track:

- **Track overview** — from track plan intent, or derived from track.yml description
- **Sandbox architecture** — ASCII diagram of hosts, ports, and connections from config.yml
- **Hosts table** — hostname, image, machine type, role for each container/VM
- **Tabs table** — tab title, type, hostname, port for each challenge's tabs
- **Challenge map** — ordered list of challenges with slug, title, difficulty, time estimate
- **Deployment instructions** — `instruqt track push`, validate, test commands
- **Troubleshooting** — common issues derived from the track's infrastructure pattern

### Step 4: Write README

Write `${TRACK_OUTPUT_DIR}/README.md`.

If a README.md already exists, prompt the user: Overwrite / Merge / Abort.

## Error Handling

| Error | Message |
|-------|---------|
| No track.yml | This doesn't look like a track directory. |
| No challenges | Generate challenges first, then generate the README. |

## Important Notes

- The README is for track authors and maintainers, not learners
- Include the sandbox architecture diagram — it is the most valuable part for onboarding new contributors
- Do not include customer-specific information (API keys, internal URLs) in the README
- The `id` field from track.yml should not be included in the README

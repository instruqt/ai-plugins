# [Track Title]

**Platform:** Instruqt
**Audience:** [Primary audience]
**Duration:** [Approximate duration]
**Difficulty:** [Basic / Intermediate / Advanced / Basic → Advanced (progressive)]
**Outcome:** [One-sentence completion payoff]
**Pausable:** [No / Yes — up to N days (pausable_ttl: N)]

> **Session notes:** build decisions, known-good state, and unresolved items for
> this track are tracked in [`MEMORY.md`](./MEMORY.md). Read it before editing.

---

## Track overview

[One to two paragraphs describing what the learner walks away able to do, what real product/workflow it mirrors, whether the backend is real or mocked, and anything special about the sandbox.]

---

## Directory structure

```
[track-slug]/
│
├── track.yml                           # Track metadata, timing, lab config
├── config.yml                          # Sandbox infrastructure (optional if sandbox_preset)
├── MEMORY.md                           # Track-level session notes (optional but recommended)
│
├── track_scripts/                      # Track-level lifecycle scripts (optional)
│   ├── setup-[hostname-1]              # Pre-challenge provisioning
│   └── setup-[hostname-2]              # (repeat per host)
│
├── assets/                             # Icons, images referenced by track.yml / assignments
│
├── 01-[challenge-slug]/
│   ├── assignment.md                   # Learner instructions (YAML frontmatter + Markdown body)
│   ├── setup-[hostname]                # Per-challenge setup (optional)
│   ├── check-[hostname]                # Validation (exit 0 = pass, exit 1 = fail)
│   └── solve-[hostname]                # Idempotent auto-solve (for skip / testing)
│
├── 02-[challenge-slug]/
│   └── …
│
└── NN-[challenge-slug]/
    └── …
```

---

## Sandbox architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  Instruqt Sandbox                                               │
│                                                                 │
│  ┌──────────────────────┐     ┌──────────────────────────────┐  │
│  │   [hostname-1]       │     │      [hostname-2]            │  │
│  │                      │     │                              │  │
│  │  • [tool-a]          │────▶│  [service-a]                 │  │
│  │  • [tool-b]          │     │  port [port-a]               │  │
│  └──────────────────────┘     └──────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Hosts

| Host | Image | Sizing | Role |
|------|-------|--------|------|
| `[hostname-1]` | `[image-1]` | [sizing] | [role] |
| `[hostname-2]` | `[image-2]` | [sizing] | [role] |

### Tabs exposed to the learner

| Tab title | Type | Points to | Notes |
|-----------|------|-----------|-------|
| [Tab 1] | `terminal` | `hostname: [hostname-1]` | Optional `workdir:` and `cmd:` |
| [Tab 2] | `service` | `hostname: [hostname-2]`, `port: [port]` | Add `custom_response_headers:` to override CSP if needed |
| [Tab 3] | `website` | `url: https://…` | URL must be HTTPS |

---

## Backend / API reference

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/health` | Health + version |
| `[METHOD]` | `[PATH]` | [Description] |

---

## Seeded data

| ID | Filename / record | Type | Source | Status on arrival |
|----|-------------------|------|--------|-------------------|
| `[id]` | `[file]` | [type] | [source] | [status] |

---

## Lab challenge map

| # | Title | Core skill | Check logic |
|---|-------|-----------|-------------|
| 1 | [Challenge 1 Title] | [Core skill] | [What check script validates] |
| 2 | [Challenge 2 Title] | [Core skill] | [What check script validates] |
| N | [Challenge N Title] | [Core skill] | [What check script validates] |

---

## Deploying the track

### Prerequisites

- Instruqt CLI installed and authenticated (`instruqt auth login`)
- Member of the `[instruqt-team-slug]` Instruqt team

### Push to Instruqt

```bash
cd [track-slug]/
instruqt track validate
instruqt track push
instruqt track open
```

### Pull for local editing

```bash
instruqt track pull [track-slug]
```

### Enable hot start

Hot start is configured in the Instruqt web UI (not via CLI):
**Settings → Hot Start** on the track page.

---

## Customisation guide

### Swap in a real backend

1. Replace the mock endpoint env var in `config.yml` → `containers.[hostname].environment`
2. Store the real API key as an Instruqt secret and reference it under `secrets:`
3. Update the relevant `setup-[hostname]` to export the secret
4. Remove the mock backend container if no longer needed
5. Update each challenge's `check-*` script to validate against real data

### Embed in your LMS

Add Instruqt's LTI integration to embed the track inside Docebo, Cornerstone,
Skilljar, or any LTI-compliant LMS.

### Push completion to Salesforce

Configure in **Settings → Integrations → Salesforce**.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `Cannot connect to [service]` | Backend container still starting | Wait ~30s; setup script polls for health |
| Service tab shows blank or "refused to frame" | Upstream CSP/X-Frame-Options blocks iframing | Add `custom_response_headers:` to strip the offending header |
| Check script shows generic error | `set -euo pipefail` in check script | Remove `set` flags — check scripts must not use pipefail |
| `instruqt track push` fails on tab URL | Non-HTTPS URL | Change to HTTPS, or use `type: service` with hostname+port |
| Runnable code block shows no Run button | Invalid modifier syntax | Use `` ```run `` or `` ```bash,run `` |

---

## Related resources

- [Instruqt docs: Configuration files](https://docs.instruqt.com/reference/cli/configuration-files)
- [Instruqt docs: Lifecycle scripts](https://docs.instruqt.com/sandboxes/lifecycle-scripts/scripting-overview)
- [Instruqt docs: Hot start](https://docs.instruqt.com/tracks/hot-start)
- [Product documentation link]

---

## Session notes

All build decisions, known-good state, and unresolved items live in
[`MEMORY.md`](./MEMORY.md) at the track root. Update it when you:

- Ship a meaningful fix
- Discover a reusable pattern
- Capture a hard-won lesson that should outlast the current session

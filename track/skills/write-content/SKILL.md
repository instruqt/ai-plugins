# Write Content

Orchestrates content generation by routing to knowledge base guidance.

## Progress reporting

Do not create your own top-level task list. The invoking command owns the user-facing task list. If this skill represents a single user-visible step in that list, your work maps to one entry. If you need finer-grained progress, update the existing task list rather than starting a new one.

## Before Writing

Load the relevant generation guides from the knowledge base using `discover-best-practices`:

### Per-Challenge Guidance
- `references/best-practices/components/assignment-content/` — step structure, code blocks, admonitions, collapsibles, variable interpolation
- `references/best-practices/components/tab-design/` — terminal, service, code, browser tab patterns
- `references/best-practices/components/notes-slides/` — pre-challenge notes and loading experience
- `references/best-practices/components/brand-voice/` — tone and voice matching (when available)

### Per-Script Guidance
- `references/best-practices/components/check-scripts/` — assertion patterns, fail-message quality, no-pipefail rule
- `references/best-practices/components/solve-scripts/` — idempotent operations, full-chain defensive patterns
- `references/best-practices/components/setup-scripts/` — bootstrap sentinels, environment config
- `references/best-practices/components/cleanup-scripts/` — resource teardown, state reset

### Cross-Challenge Guidance
- `references/best-practices/holistic/educational-progression/` — scaffolding, progressive complexity
- `references/best-practices/holistic/content-coherence/` — terminology consistency, naming conventions
- `references/best-practices/holistic/learner-experience/` — cognitive load, pacing

## Loading Strategy

**Load only what you need for the current generation step.** Do not pre-load all categories listed above. Use `discover-best-practices` to query for specific guidance, then read only the matched files.

| Current step | Query for | Skip |
|---|---|---|
| Writing assignment markdown | assignment-content, tab-design, brand-voice | all script categories, holistic |
| Writing check/solve scripts | check-scripts, solve-scripts | assignment-content, setup-scripts, holistic |
| Writing setup scripts | setup-scripts, cleanup-scripts | assignment-content, check/solve scripts, holistic |
| Reviewing challenge progression | educational-progression, content-coherence | all component categories |

**Hard rule:** Do not load holistic best-practices during single-file generation. Holistic guidance (educational progression, content coherence, learner experience) is relevant only when evaluating cross-challenge quality — not when writing one file. The scorer agents handle holistic evaluation separately.

## Assignment Body Structure

The assignment.md body uses Markdown with Instruqt extensions:
- Numbered steps with clear instructions
- Code blocks with modifiers (`run`, `copy`, `nocopy`)
- Tab-switching buttons referencing tab indices (`[tab-N]`)
- Variable interpolation (`[[ Instruqt-Var ]]`)
- Admonitions (NOTE, TIP, IMPORTANT, WARNING)
- Collapsible hints via `<details><summary>`

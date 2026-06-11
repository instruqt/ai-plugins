# Design Challenges

Guide challenge sequencing, difficulty progression, and infrastructure planning for a track.

## Progress reporting

Do not create your own top-level task list. The invoking command owns the user-facing task list. If this skill represents a single user-visible step in that list, your work maps to one entry. If you need finer-grained progress, update the existing task list rather than starting a new one.

## Before Designing

Load relevant guidance using `discover-best-practices`:
- `references/best-practices/components/challenge-structure/` — numbering, ordering, slug naming, difficulty
- `references/best-practices/holistic/educational-progression/` — progressive complexity, scaffolding, concept islands
- `references/best-practices/holistic/functional-integrity/` — setup-challenge alignment, script-assignment alignment
- `references/best-practices/holistic/learner-experience/` — pacing, cognitive load, target audience

Load infrastructure patterns using `discover-examples`:
- `references/infrastructure/` — sandbox architecture options
- `references/decision-frameworks/` — trade-off guidance for infrastructure choices

## Challenge Sequencing Principles

1. **Observation → Guided → Independent** arc across the track
2. Each challenge builds on the previous — no concept islands
3. Early challenges introduce vocabulary; later challenges use it
4. Time estimates account for reading + doing + potential mistakes
5. One core concept per challenge, reinforced through action

## Infrastructure Awareness

When designing challenges, consider what infrastructure changes each needs:
- Does this challenge need a new service running? → setup script
- Does this challenge need to validate learner work? → check script assertions
- Does this challenge modify state that the next challenge depends on? → solve script ensures correct state
- Does this challenge need cleanup before the next? → cleanup script

## Script Planning Per Challenge

For each challenge, define:
- **Setup:** What environment state must be present before the learner starts
- **Check:** What specific assertions validate the learner completed the task correctly
- **Solve:** What commands produce the exact state that passes the check (must be idempotent)
- **Cleanup:** What state needs resetting (only if needed between challenges)

## Time Budgeting

- Short challenges (5-10 min): single concept, 2-4 steps
- Medium challenges (10-20 min): connected concepts, 5-8 steps
- Long challenges (20-30 min): complex multi-step tasks (avoid when possible)
- Total track time: typically 30-90 minutes

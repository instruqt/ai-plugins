# Discover Best Practices

Find relevant best-practice and anti-pattern files for a given query.

## Progress reporting

Do not create your own top-level task list. The invoking command owns the user-facing task list. If this skill represents a single user-visible step in that list, your work maps to one entry. If you need finer-grained progress, update the existing task list rather than starting a new one.

## Paths

- Manifest: `${CLAUDE_PLUGIN_ROOT}/references/best-practices/_index.md`

## Workflow

1. Read the manifest file
2. Match the query against file descriptions — use keyword overlap, synonyms, and related concepts
3. Return matching file paths with their one-line descriptions
4. The invoking agent reads the files it needs

## Scope

This skill covers:
- Component best practices (`references/best-practices/components/`) — per-file quality criteria
- Holistic best practices (`references/best-practices/holistic/`) — cross-challenge quality criteria
- Anti-patterns (`references/best-practices/anti-patterns/`) — known mistakes to avoid

## Usage

Agent invokes: "What best practices apply to check scripts?"
Skill returns:
```
- components/check-scripts/assertion-patterns.md — Diagnostic sequencing of independent assertions
- components/check-scripts/fail-message-quality.md — Actionable messages with remediation steps
- components/check-scripts/no-pipefail-rule.md — No set -euo pipefail in check scripts
- components/check-scripts/timeout-awareness.md — 60-second hard limit, time budgeting
- components/check-scripts/idempotency.md — Side-effect-free, read-only operations
- anti-patterns/commented-out-set-flags.md — Commented-out set flags in scripts
- anti-patterns/missing-o-pipefail.md — Missing -o pipefail where required
```

Agent invokes: "What holistic criteria apply when reviewing challenge progression?"
Skill returns:
```
- holistic/educational-progression/progressive-complexity.md — Complexity ramp across challenges
- holistic/educational-progression/scaffolding-pattern.md — Guided to independent progression
- holistic/educational-progression/builds-on-prior-knowledge.md — Each challenge builds on previous
- holistic/content-coherence/challenge-dependencies.md — Challenge dependency chain
```

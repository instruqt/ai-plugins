# Evaluation Reference Index

- `scoring-guide.md` — Scoring guide: global score-level definitions (0-1 checklist, 1-5 analytic/holistic), calibration for scorer agents
- `scorer-prompts.md` — Scorer prompt templates (analytic/holistic/checklist) the orchestrating agent uses to build each scorer sub-agent's prompt

## Analytic: Plan Track

- `analytic/plan-track/learning-objectives.md` — Learning objectives: clarity, measurability, alignment with track intent and audience
- `analytic/plan-track/audience-definition.md` — Audience definition: specificity of target audience, prerequisites, skill level assumptions
- `analytic/plan-track/challenge-roadmap.md` — Challenge roadmap: challenge sequencing, timelimits, difficulty progression, logical flow
- `analytic/plan-track/infrastructure-design.md` — Infrastructure design: config.yml resources, quota compliance, machine type appropriateness

## Analytic: Plan Challenge

- `analytic/plan-challenge/assignment-flow.md` — Assignment flow: step sequencing, progressive disclosure, learner guidance within a single challenge
- `analytic/plan-challenge/builds-on-prior.md` — Builds on prior: challenge connects to and extends what earlier challenges taught
- `analytic/plan-challenge/infrastructure-changes.md` — Infrastructure changes: per-challenge setup/cleanup needs, config.yml amendments
- `analytic/plan-challenge/script-design.md` — Script design: check/solve/setup script design quality, convention adherence
- `analytic/plan-challenge/time-estimates.md` — Time estimates: per-challenge duration realism, cumulative track time, complexity-duration alignment

## Analytic: Generate

- `analytic/generate/assignment-content.md` — Assignment content: markdown quality, code blocks, step structure, tab references
- `analytic/generate/brand-voice.md` — Brand voice: tone, style, terminology consistency with customer writing style
- `analytic/generate/challenge-design.md` — Challenge design: goal clarity, scoping, step structure, check alignment, title quality
- `analytic/generate/config-completeness.md` — Config completeness: config.yml resources match what the track plan requires
- `analytic/generate/interactivity.md` — Interactivity: concept reinforcement through practice, check feedback quality, tab support, progressive independence
- `analytic/generate/script-assignment-alignment.md` — Script-assignment alignment: check/solve scripts validate exactly what the assignment asks learners to do
- `analytic/generate/script-quality.md` — Script quality: bash conventions, error handling, idempotency, platform awareness
- `analytic/generate/tab-layout.md` — Tab layout: tab types, ordering, hostname/port correctness, learner workflow
- `analytic/generate/track-metadata.md` — Track metadata: title, description, tags, timing settings, lab_config appropriateness
- `analytic/generate/track-structure.md` — Track structure: challenge ordering, balance, difficulty progression, directory conventions, config coherence

## Checklist: Generate

- `checklist/generate/completeness.md` — Completeness: all required files present, scripts executable, no missing artifacts
- `checklist/generate/conventions.md` — Conventions: shebangs, set flags, naming patterns, file structure compliance
- `checklist/generate/validation.md` — Validation: shellcheck passes, YAML valid, quotas respected, ports aligned

## Holistic

- `holistic/plan-track/learning-coherence.md` — Learning coherence: track-level gestalt quality, objectives-to-challenges alignment
- `holistic/plan-challenge/challenge-coherence.md` — Challenge coherence: challenge-level gestalt quality, assignment-scripts-tabs alignment

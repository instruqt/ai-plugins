# Best Practices Reference Index

## Assignment Content

- `components/assignment-content/step-structure.md` — Step-based body structure: numbered steps, section separation, completion markers in assignment markdown
- `components/assignment-content/code-blocks.md` — Code block usage: correct modifiers, language tags, placeholder handling, run/copy behavior
- `components/assignment-content/admonitions.md` — Callout and admonition quality: appropriate type selection, usage frequency, information highlighting
- `components/assignment-content/collapsibles-and-hints.md` — Collapsible hint sections: opt-in help without spoiling answers, progressive disclosure
- `components/assignment-content/tab-buttons.md` — Tab button usage: correct tab index references, semantic button variants, visual hierarchy
- `components/assignment-content/variable-interpolation.md` — Variable interpolation quality: dynamic value rendering, no raw env var names or stale placeholders
- `components/assignment-content/inline-html.md` — Inline HTML usage: purposeful HTML where markdown falls short, accessible and consistent styling

## Challenge Structure

- `components/challenge-structure/slug-naming.md` — Slug naming conventions: descriptive, consistent slugs readable as a table of contents
- `components/challenge-structure/numbering-conventions.md` — Numbering conventions: sequential directory numbering aligned with track.yml ordering
- `components/challenge-structure/ordering-and-progression.md` — Challenge ordering: logical learning flow with progressive difficulty, no concept jumps
- `components/challenge-structure/difficulty-progression.md` — Difficulty field accuracy: challenge difficulty values matching content complexity, progressive ramp
- `components/challenge-structure/timelimit-settings.md` — Per-challenge timelimit: appropriate values for complexity, alignment with track-level timing

## Check Scripts

- `components/check-scripts/assertion-patterns.md` — Check script assertion patterns: validation quality for verifying learner progress
- `components/check-scripts/common-check-patterns.md` — Common check patterns catalog: reusable validation patterns for file existence, content, services, API responses
- `components/check-scripts/fail-message-quality.md` — Fail-message quality: actionable, clear text shown to learners when check assertions fail
- `components/check-scripts/idempotency.md` — Check script idempotency: side-effect-free validation, safe to run multiple times
- `components/check-scripts/no-pipefail-rule.md` — No pipefail in check scripts: must NOT use set -euo pipefail (opposite of setup/solve scripts)
- `components/check-scripts/timeout-awareness.md` — Check script timeout: hard 60-second platform limit, avoiding long-running commands
- `components/check-scripts/image-dependent-helpers.md` — Image-dependent helpers: fail-message and set-status commands depend on sandbox image availability

## Cleanup Scripts

- `components/cleanup-scripts/cost-management.md` — Cleanup cost management: preventing cost leakage by deleting all billable external resources
- `components/cleanup-scripts/resource-teardown.md` — Resource teardown: properly deleting external and costed resources provisioned during setup or execution
- `components/cleanup-scripts/state-reset.md` — State reset between challenges: per-challenge cleanup providing clean slate for next challenge

## Config.yml

- `components/config-yml/cloud-account-configuration.md` — Cloud account IAM configuration: AWS/Azure/GCP resource blocks, policies, services, region settings
- `components/config-yml/container-image-selection.md` — Container and VM image selection: balancing pre-installed tooling against image size and startup time
- `components/config-yml/cross-vm-networking.md` — Cross-VM networking: inter-host communication, hostname discovery, state sharing, file transfer
- `components/config-yml/port-exposure.md` — Port exposure configuration: external ingress settings, minimizing unnecessary service exposure
- `components/config-yml/resource-sizing.md` — Resource sizing: VM and container memory/CPU allocation balancing cost efficiency and learner experience
- `components/config-yml/secrets-management.md` — Secrets management: API keys, tokens, SSH keys via secrets block, preventing leaks through env/scripts/images

## Notes and Slides

- `components/notes-slides/notes-content-patterns.md` — Notes slide content patterns: effective patterns for different challenge positions and track contexts
- `components/notes-slides/pre-challenge-notes.md` — Pre-challenge notes slide design: learner context and engagement during challenge loading
- `components/notes-slides/video-and-image-slides.md` — Video and image slides: correct formats, sizing, meaningful visual context

## Setup Scripts

- `components/setup-scripts/track-vs-challenge-setup.md` — Track-level vs per-challenge setup placement: one-time sandbox init vs per-challenge configuration
- `components/setup-scripts/background-parallelism.md` — Background parallelism: running independent slow operations in parallel to minimize setup time
- `components/setup-scripts/bootstrap-sentinels.md` — Bootstrap sentinel waiting: ensuring host bootstrap completes before setup logic runs
- `components/setup-scripts/debian-frontend.md` — DEBIAN_FRONTEND and package management: suppressing interactive prompts, preventing setup timeouts
- `components/setup-scripts/environment-configuration.md` — Environment configuration: shell env vars, prompt customization, working directory, completions
- `components/setup-scripts/hot-start-awareness.md` — Hot start awareness: handling hot-start and invite-link scenarios with empty user-specific env vars
- `components/setup-scripts/tool-installation.md` — Tool installation: version pinning, platform detection, speed optimization in setup scripts
- `components/setup-scripts/readiness-patterns.md` — Readiness patterns: retry helpers, exponential backoff, jitter, HTTP status polling, dual probes for service availability
- `components/setup-scripts/debugging-patterns.md` — Debugging patterns: PS4 line-numbered xtrace, ERR traps, banner-echo for log navigation in sandboxes

## Solve Scripts

- `components/solve-scripts/check-expected-state.md` — Check-expected state: solve must produce the exact state that check validates, not approximate
- `components/solve-scripts/full-chain-defensive.md` — Full-chain defensive: solve scripts recreate entire prerequisite chain, safe for skipping_enabled
- `components/solve-scripts/idempotent-operations.md` — Idempotent solve operations: safe to re-run, same end state on repeated execution
- `components/solve-scripts/set-flags.md` — Solve script set flags: set -euxo pipefail required (opposite of check scripts)

## Tab Design

- `components/tab-design/terminal-tabs.md` — Terminal tab design: user context, working directory, startup commands for ready-to-use shell
- `components/tab-design/service-tabs.md` — Service tab design: web UI exposure, hostname/port mapping, deep linking, CSP handling for iframe embedding
- `components/tab-design/browser-tabs.md` — Browser and website tab design: correct type, URL scheme, variable interpolation for cloud consoles
- `components/tab-design/code-editor-tabs.md` — Code editor tab design: file/directory targeting for effective learner editing experience
- `components/tab-design/csp-header-overrides.md` — CSP and X-Frame-Options overrides: handling headers for web UIs inside the Instruqt player iframe

## Track.yml

- `components/track-yml/metadata-quality.md` — Metadata quality: title, teaser, description clarity and catalog discovery optimization
- `components/track-yml/timing-settings.md` — Timing settings: timelimit and idle_timeout configuration for track scope, audience, delivery context
- `components/track-yml/lab-config.md` — Lab config: lab_config sub-fields for optimal layout and interaction experience
- `components/track-yml/loading-experience.md` — Loading experience: enhanced_loading, notes slides, loadingMessages to engage learners during provisioning
- `components/track-yml/theming-and-branding.md` — Theming and branding: theme, icon, description HTML, loading messages for cohesive branded experience

## Content Coherence

- `holistic/content-coherence/terminology-consistency.md` — Terminology consistency: same concept uses same term throughout the track
- `holistic/content-coherence/technical-terms-introduced-before-use.md` — Technical terms introduced before use: plain-language explanation on first appearance
- `holistic/content-coherence/cross-references.md` — Cross-references: consistent references to earlier material without contradictions
- `holistic/content-coherence/challenge-dependencies.md` — Challenge dependencies: each challenge depends only on concepts taught in prior challenges
- `holistic/content-coherence/file-and-resource-naming.md` — File and resource naming: consistent naming across YAML config, scripts, assignments, directories
- `holistic/content-coherence/no-accidental-future-task-spoilers.md` — No accidental future-challenge spoilers: examples must not pre-complete later challenge steps

## Educational Progression

- `holistic/educational-progression/progressive-complexity.md` — Progressive complexity: early challenges provide more guidance, later expect more independence
- `holistic/educational-progression/scaffolding-pattern.md` — Scaffolding pattern: concepts progress from modeled to guided to independent
- `holistic/educational-progression/builds-on-prior-knowledge.md` — Builds on prior knowledge: later content references and builds on earlier material
- `holistic/educational-progression/no-concept-islands.md` — No concept islands: every concept connects to something before and after, no isolated teaching
- `holistic/educational-progression/active-learning-throughout.md` — Active learning throughout: learners doing at every stage, not just reading, meaningful interaction

## Functional Integrity

- `holistic/functional-integrity/end-to-end-task-flow.md` — End-to-end task flow: every challenge completable in sequence without blockers
- `holistic/functional-integrity/script-assignment-alignment.md` — Script-assignment alignment: check script validates what assignment.md actually asks learners to do
- `holistic/functional-integrity/setup-challenge-alignment.md` — Setup-challenge alignment: environment in correct state when challenge begins, scripts consistent

## Learner Experience

- `holistic/learner-experience/appropriate-for-target-audience.md` — Appropriate for target audience: terminology, explanations, complexity match stated skill level
- `holistic/learner-experience/cognitive-load-management.md` — Cognitive load management: avoiding information overload across challenges
- `holistic/learner-experience/no-assumed-knowledge-beyond-prerequisites.md` — No assumed knowledge beyond prerequisites: content stays within stated prerequisite boundaries
- `holistic/learner-experience/pacing.md` — Pacing: consistent challenge pacing for engagement without fatigue, cross-challenge rhythm
- `holistic/learner-experience/recoverable-from-mistakes.md` — Recoverable from mistakes: learners can recover from errors without getting permanently stuck
- `holistic/learner-experience/user-autonomy.md` — User autonomy: written context and troubleshooting notes match the delivery context's autonomy level

## Anti-Patterns

- `anti-patterns/bake-credentials-into-images.md` — Baking credentials into images: embedding long-lived API keys, tokens, private keys in custom images
- `anti-patterns/base64-docker-credentials.md` — Base64 Docker credentials: hardcoding dockerconfigjson as base64 strings in setup scripts
- `anti-patterns/certbot-at-runtime.md` — Certbot at runtime: running certbot when provision_ssl_certificate: true is available
- `anti-patterns/commented-out-set-flags.md` — Commented-out set flags: #set -euxo pipefail missing space or intended as active code
- `anti-patterns/hardcode-github-pats.md` — Hardcoded GitHub PATs: Personal Access Tokens or repo tokens directly in lifecycle scripts
- `anti-patterns/invalid-gcp-role.md` — Invalid GCP role: using non-existent roles/writer as a GCP IAM role
- `anti-patterns/lolcat-in-lifecycle-scripts.md` — Lolcat in lifecycle scripts: ANSI escape tools (lolcat, cowsay, ponysay) producing garbled output
- `anti-patterns/missing-cloud-resource-block.md` — Missing cloud resource block: referencing INSTRUQT_AWS/AZURE/GCP env vars without matching config.yml resource
- `anti-patterns/missing-equals-debian-frontend.md` — Missing equals in DEBIAN_FRONTEND: export DEBIAN_FRONTEND noninteractive without = sign
- `anti-patterns/missing-o-pipefail.md` — Missing -o in pipefail: set -eux pipefail instead of set -euxo pipefail
- `anti-patterns/nohup-across-challenges.md` — Nohup across challenges: relying on nohup for background processes across challenge boundaries
- `anti-patterns/python-run-for-non-python.md` — Python run for non-Python: using python,run code fence for content that is not Python code
- `anti-patterns/secrets-in-environment-block.md` — Secrets in environment block: SSH keys, API keys, secrets in config.yml environment: block
- `anti-patterns/skip-cleanup-for-costed-resources.md` — Skipping cleanup for costed resources: omitting cleanup scripts for external billable resources
- `anti-patterns/sleep-instead-of-polling.md` — Sleep instead of polling: using sleep N as a substitute for infrastructure readiness polling
- `anti-patterns/socat-for-iframe-headers.md` — Socat for iframe headers: using socat to fix X-Frame-Options or Content-Security-Policy headers
- `anti-patterns/vestigial-nginx-zones.md` — Vestigial nginx-zones container: copying unused nginx-zones from Consul HVD configs into new tracks

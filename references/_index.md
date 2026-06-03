# Reference Index

## Track Format

- `track-format/track-yml.md` — track.yml structure: track identity, timing, UI configuration, ownership fields
- `track-format/config-yml.md` — config.yml structure: sandbox infrastructure definition for containers, VMs, virtual browsers, cloud accounts, secrets
- `track-format/assignment-md.md` — assignment.md structure: challenge YAML frontmatter (metadata, tabs) and Markdown body for learner instructions
- `track-format/lifecycle-scripts.md` — Lifecycle script types (setup, check, solve, cleanup), execution order, bash/powershell, naming conventions
- `track-format/tabs.md` — Tab configuration in assignment.md frontmatter: terminal, service, code, browser, website, and external tab types
- `track-format/containers.md` — Container resource definition in config.yml: image, ports, environment, memory, shell, entrypoint
- `track-format/virtualmachines.md` — Virtual machine resource definition in config.yml: image, machine_type, memory, cpus, nested virtualization
- `track-format/virtualbrowsers.md` — Virtual browser definition in config.yml: embedded browser tabs for cloud console sign-in pages
- `track-format/cloud-accounts.md` — Cloud account configuration (AWS, Azure, GCP) in config.yml: dual-role IAM, learner vs admin permissions
- `track-format/secrets.md` — Secrets block in config.yml: named secrets injected as environment variables, never committed to source
- `track-format/sandbox-presets.md` — Sandbox presets: pre-built environments replacing config.yml, no track_scripts directory needed
- `track-format/custom-resources.md` — Custom resources: Terraform modules provisioned as sandbox resources before other components
- `track-format/machine-types.md` — GCP machine types for VM sizing: machine_type field vs memory/cpus, available instance families
- `track-format/quotas-and-limits.md` — Platform quotas and resource limits: sandbox host constraints, cloud project limits, script timeouts
- `track-format/variable-interpolation.md` — Variable interpolation: agent CLI variable setting in scripts, shortcode syntax in markdown
- `track-format/code-block-modifiers.md` — Code block modifiers: auto-run, copy button, line wrap, line numbers appended to fenced code blocks
- `track-format/buttons-and-navigation.md` — Inline buttons and navigation: tab switching, section navigation, external links, admonitions, collapsibles
- `track-format/ui-checks.md` — UI checks: virtual browser DOM element inspection and URL validation complementing lifecycle check scripts

## Component Best Practices

### Assignment Content

- `best-practices/components/assignment-content/step-structure.md` — Step-based body structure: numbered steps, section separation, completion markers in assignment markdown
- `best-practices/components/assignment-content/code-blocks.md` — Code block usage: correct modifiers, language tags, placeholder handling, run/copy behavior
- `best-practices/components/assignment-content/admonitions.md` — Callout and admonition quality: appropriate type selection, usage frequency, information highlighting
- `best-practices/components/assignment-content/collapsibles-and-hints.md` — Collapsible hint sections: opt-in help without spoiling answers, progressive disclosure
- `best-practices/components/assignment-content/tab-buttons.md` — Tab button usage: correct tab index references, semantic button variants, visual hierarchy
- `best-practices/components/assignment-content/variable-interpolation.md` — Variable interpolation quality: dynamic value rendering, no raw env var names or stale placeholders
- `best-practices/components/assignment-content/inline-html.md` — Inline HTML usage: purposeful HTML where markdown falls short, accessible and consistent styling

### Challenge Structure

- `best-practices/components/challenge-structure/slug-naming.md` — Slug naming conventions: descriptive, consistent slugs readable as a table of contents
- `best-practices/components/challenge-structure/numbering-conventions.md` — Numbering conventions: sequential directory numbering aligned with track.yml ordering
- `best-practices/components/challenge-structure/ordering-and-progression.md` — Challenge ordering: logical learning flow with progressive difficulty, no concept jumps
- `best-practices/components/challenge-structure/difficulty-progression.md` — Difficulty field accuracy: challenge difficulty values matching content complexity, progressive ramp
- `best-practices/components/challenge-structure/timelimit-settings.md` — Per-challenge timelimit: appropriate values for complexity, alignment with track-level timing

### Check Scripts

- `best-practices/components/check-scripts/assertion-patterns.md` — Check script assertion patterns: validation quality for verifying learner progress
- `best-practices/components/check-scripts/common-check-patterns.md` — Common check patterns catalog: reusable validation patterns for file existence, content, services, API responses
- `best-practices/components/check-scripts/fail-message-quality.md` — Fail-message quality: actionable, clear text shown to learners when check assertions fail
- `best-practices/components/check-scripts/idempotency.md` — Check script idempotency: side-effect-free validation, safe to run multiple times
- `best-practices/components/check-scripts/no-pipefail-rule.md` — No pipefail in check scripts: must NOT use set -euo pipefail (opposite of setup/solve scripts)
- `best-practices/components/check-scripts/timeout-awareness.md` — Check script timeout: hard 60-second platform limit, avoiding long-running commands
- `best-practices/components/check-scripts/image-dependent-helpers.md` — Image-dependent helpers: fail-message and set-status commands depend on sandbox image availability

### Cleanup Scripts

- `best-practices/components/cleanup-scripts/cost-management.md` — Cleanup cost management: preventing cost leakage by deleting all billable external resources
- `best-practices/components/cleanup-scripts/resource-teardown.md` — Resource teardown: properly deleting external and costed resources provisioned during setup or execution
- `best-practices/components/cleanup-scripts/state-reset.md` — State reset between challenges: per-challenge cleanup providing clean slate for next challenge

### Config.yml

- `best-practices/components/config-yml/cloud-account-configuration.md` — Cloud account IAM configuration: AWS/Azure/GCP resource blocks, policies, services, region settings
- `best-practices/components/config-yml/container-image-selection.md` — Container and VM image selection: balancing pre-installed tooling against image size and startup time
- `best-practices/components/config-yml/cross-vm-networking.md` — Cross-VM networking: inter-host communication, hostname discovery, state sharing, file transfer
- `best-practices/components/config-yml/port-exposure.md` — Port exposure configuration: external ingress settings, minimizing unnecessary service exposure
- `best-practices/components/config-yml/resource-sizing.md` — Resource sizing: VM and container memory/CPU allocation balancing cost efficiency and learner experience
- `best-practices/components/config-yml/secrets-management.md` — Secrets management: API keys, tokens, SSH keys via secrets block, preventing leaks through env/scripts/images

### Notes and Slides

- `best-practices/components/notes-slides/notes-content-patterns.md` — Notes slide content patterns: effective patterns for different challenge positions and track contexts
- `best-practices/components/notes-slides/pre-challenge-notes.md` — Pre-challenge notes slide design: learner context and engagement during challenge loading
- `best-practices/components/notes-slides/video-and-image-slides.md` — Video and image slides: correct formats, sizing, meaningful visual context

### Setup Scripts

- `best-practices/components/setup-scripts/track-vs-challenge-setup.md` — Track-level vs per-challenge setup placement: one-time sandbox init vs per-challenge configuration
- `best-practices/components/setup-scripts/background-parallelism.md` — Background parallelism: running independent slow operations in parallel to minimize setup time
- `best-practices/components/setup-scripts/bootstrap-sentinels.md` — Bootstrap sentinel waiting: ensuring host bootstrap completes before setup logic runs
- `best-practices/components/setup-scripts/debian-frontend.md` — DEBIAN_FRONTEND and package management: suppressing interactive prompts, preventing setup timeouts
- `best-practices/components/setup-scripts/environment-configuration.md` — Environment configuration: shell env vars, prompt customization, working directory, completions
- `best-practices/components/setup-scripts/hot-start-awareness.md` — Hot start awareness: handling hot-start and invite-link scenarios with empty user-specific env vars
- `best-practices/components/setup-scripts/tool-installation.md` — Tool installation: version pinning, platform detection, speed optimization in setup scripts

### Solve Scripts

- `best-practices/components/solve-scripts/check-expected-state.md` — Check-expected state: solve must produce the exact state that check validates, not approximate
- `best-practices/components/solve-scripts/full-chain-defensive.md` — Full-chain defensive: solve scripts recreate entire prerequisite chain, safe for skipping_enabled
- `best-practices/components/solve-scripts/idempotent-operations.md` — Idempotent solve operations: safe to re-run, same end state on repeated execution
- `best-practices/components/solve-scripts/set-flags.md` — Solve script set flags: set -euxo pipefail required (opposite of check scripts)

### Tab Design

- `best-practices/components/tab-design/terminal-tabs.md` — Terminal tab design: user context, working directory, startup commands for ready-to-use shell
- `best-practices/components/tab-design/service-tabs.md` — Service tab design: web UI exposure, hostname/port mapping, deep linking, CSP handling for iframe embedding
- `best-practices/components/tab-design/browser-tabs.md` — Browser and website tab design: correct type, URL scheme, variable interpolation for cloud consoles
- `best-practices/components/tab-design/code-editor-tabs.md` — Code editor tab design: file/directory targeting for effective learner editing experience
- `best-practices/components/tab-design/csp-header-overrides.md` — CSP and X-Frame-Options overrides: handling headers for web UIs inside the Instruqt player iframe

### Track.yml

- `best-practices/components/track-yml/metadata-quality.md` — Metadata quality: title, teaser, description clarity and catalog discovery optimization
- `best-practices/components/track-yml/timing-settings.md` — Timing settings: timelimit and idle_timeout configuration for track scope, audience, delivery context
- `best-practices/components/track-yml/lab-config.md` — Lab config: lab_config sub-fields for optimal layout and interaction experience
- `best-practices/components/track-yml/loading-experience.md` — Loading experience: enhanced_loading, notes slides, loadingMessages to engage learners during provisioning
- `best-practices/components/track-yml/theming-and-branding.md` — Theming and branding: theme, icon, description HTML, loading messages for cohesive branded experience

## Holistic Best Practices

### Content Coherence

- `best-practices/holistic/content-coherence/terminology-consistency.md` — Terminology consistency: same concept uses same term throughout the track
- `best-practices/holistic/content-coherence/technical-terms-introduced-before-use.md` — Technical terms introduced before use: plain-language explanation on first appearance
- `best-practices/holistic/content-coherence/cross-references.md` — Cross-references: consistent references to earlier material without contradictions
- `best-practices/holistic/content-coherence/challenge-dependencies.md` — Challenge dependencies: each challenge depends only on concepts taught in prior challenges
- `best-practices/holistic/content-coherence/file-and-resource-naming.md` — File and resource naming: consistent naming across YAML config, scripts, assignments, directories
- `best-practices/holistic/content-coherence/no-accidental-future-task-spoilers.md` — No accidental future-challenge spoilers: examples must not pre-complete later challenge steps

### Educational Progression

- `best-practices/holistic/educational-progression/progressive-complexity.md` — Progressive complexity: early challenges provide more guidance, later expect more independence
- `best-practices/holistic/educational-progression/scaffolding-pattern.md` — Scaffolding pattern: concepts progress from modeled to guided to independent
- `best-practices/holistic/educational-progression/builds-on-prior-knowledge.md` — Builds on prior knowledge: later content references and builds on earlier material
- `best-practices/holistic/educational-progression/no-concept-islands.md` — No concept islands: every concept connects to something before and after, no isolated teaching
- `best-practices/holistic/educational-progression/active-learning-throughout.md` — Active learning throughout: learners doing at every stage, not just reading, meaningful interaction

### Functional Integrity

- `best-practices/holistic/functional-integrity/end-to-end-task-flow.md` — End-to-end task flow: every challenge completable in sequence without blockers
- `best-practices/holistic/functional-integrity/script-assignment-alignment.md` — Script-assignment alignment: check script validates what assignment.md actually asks learners to do
- `best-practices/holistic/functional-integrity/setup-challenge-alignment.md` — Setup-challenge alignment: environment in correct state when challenge begins, scripts consistent

### Learner Experience

- `best-practices/holistic/learner-experience/appropriate-for-target-audience.md` — Appropriate for target audience: terminology, explanations, complexity match stated skill level
- `best-practices/holistic/learner-experience/cognitive-load-management.md` — Cognitive load management: avoiding information overload across challenges
- `best-practices/holistic/learner-experience/no-assumed-knowledge-beyond-prerequisites.md` — No assumed knowledge beyond prerequisites: content stays within stated prerequisite boundaries
- `best-practices/holistic/learner-experience/pacing.md` — Pacing: consistent challenge pacing for engagement without fatigue, cross-challenge rhythm
- `best-practices/holistic/learner-experience/recoverable-from-mistakes.md` — Recoverable from mistakes: learners can recover from errors without getting permanently stuck
- `best-practices/holistic/learner-experience/user-autonomy.md` — User autonomy: written context and troubleshooting notes match the delivery context's autonomy level

## Anti-Patterns

- `best-practices/anti-patterns/bake-credentials-into-images.md` — Baking credentials into images: embedding long-lived API keys, tokens, private keys in custom images
- `best-practices/anti-patterns/base64-docker-credentials.md` — Base64 Docker credentials: hardcoding dockerconfigjson as base64 strings in setup scripts
- `best-practices/anti-patterns/certbot-at-runtime.md` — Certbot at runtime: running certbot when provision_ssl_certificate: true is available
- `best-practices/anti-patterns/commented-out-set-flags.md` — Commented-out set flags: #set -euxo pipefail missing space or intended as active code
- `best-practices/anti-patterns/hardcode-github-pats.md` — Hardcoded GitHub PATs: Personal Access Tokens or repo tokens directly in lifecycle scripts
- `best-practices/anti-patterns/invalid-gcp-role.md` — Invalid GCP role: using non-existent roles/writer as a GCP IAM role
- `best-practices/anti-patterns/lolcat-in-lifecycle-scripts.md` — Lolcat in lifecycle scripts: ANSI escape tools (lolcat, cowsay, ponysay) producing garbled output
- `best-practices/anti-patterns/missing-cloud-resource-block.md` — Missing cloud resource block: referencing INSTRUQT_AWS/AZURE/GCP env vars without matching config.yml resource
- `best-practices/anti-patterns/missing-equals-debian-frontend.md` — Missing equals in DEBIAN_FRONTEND: export DEBIAN_FRONTEND noninteractive without = sign
- `best-practices/anti-patterns/missing-o-pipefail.md` — Missing -o in pipefail: set -eux pipefail instead of set -euxo pipefail
- `best-practices/anti-patterns/nohup-across-challenges.md` — Nohup across challenges: relying on nohup for background processes across challenge boundaries
- `best-practices/anti-patterns/python-run-for-non-python.md` — Python run for non-Python: using python,run code fence for content that is not Python code
- `best-practices/anti-patterns/secrets-in-environment-block.md` — Secrets in environment block: SSH keys, API keys, secrets in config.yml environment: block
- `best-practices/anti-patterns/skip-cleanup-for-costed-resources.md` — Skipping cleanup for costed resources: omitting cleanup scripts for external billable resources
- `best-practices/anti-patterns/sleep-instead-of-polling.md` — Sleep instead of polling: using sleep N as a substitute for infrastructure readiness polling
- `best-practices/anti-patterns/socat-for-iframe-headers.md` — Socat for iframe headers: using socat to fix X-Frame-Options or Content-Security-Policy headers
- `best-practices/anti-patterns/vestigial-nginx-zones.md` — Vestigial nginx-zones container: copying unused nginx-zones from Consul HVD configs into new tracks

## Infrastructure

See `infrastructure/_index.md` for the full infrastructure reference covering base compute patterns, cloud accounts, managed Kubernetes, runtime provisioning, learner UX, modifiers, and sandbox presets.

## Decision Frameworks

- `decision-frameworks/agent-variable-vs-json-broker.md` — Agent variable vs JSON broker vs cross-VM env passthrough: choosing state-sharing mechanism for multi-VM tracks
- `decision-frameworks/custom-image-vs-heredoc.md` — Custom image vs heredoc/setup script: baking dependencies into Packer images vs inline generation
- `decision-frameworks/engagement-validation.md` — Engagement validation: when and how to detect genuine learner progress vs click-through gaming
- `decision-frameworks/machine-type-vs-memory-cpus.md` — Machine type vs memory/cpus: VM sizing using machine_type field vs separate memory + cpus fields
- `decision-frameworks/preset-vs-custom-config.md` — Preset vs custom config: sandbox preset vs writing a custom config.yml for track infrastructure

## Errors

- `errors/config-errors.md` — Config errors: YAML validation errors, resource misconfigurations in track config files
- `errors/push-validation-errors.md` — Push validation errors: failure patterns during instruqt track push (UTF-8, encoding, structure)
- `errors/script-errors.md` — Script errors: setup/check/solve script failures, debugging patterns, apt-get freezes, timeouts

## Evaluation

- `evaluation/scoring-guide.md` — Scoring guide: global score-level definitions (0-1 checklist, 1-5 analytic/holistic), calibration for scorer agents

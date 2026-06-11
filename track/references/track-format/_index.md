# Track Format Reference Index

- `track-yml.md` — track.yml structure: track identity, timing, UI configuration, ownership fields
- `config-yml.md` — config.yml structure: sandbox infrastructure definition for containers, VMs, virtual browsers, cloud accounts, secrets
- `assignment-md.md` — assignment.md structure: challenge YAML frontmatter (metadata, tabs) and Markdown body for learner instructions
- `lifecycle-scripts.md` — Lifecycle script types (setup, check, solve, cleanup), execution order, bash/powershell, naming conventions, helper commands, built-in env vars
- `tabs.md` — Tab configuration in assignment.md frontmatter: terminal, service, code, browser, website, and external tab types
- `containers.md` — Container resource definition in config.yml: image, ports, environment, memory, shell, entrypoint
- `virtualmachines.md` — Virtual machine resource definition in config.yml: image, machine_type, memory, cpus, nested virtualization, Windows VMs
- `virtualbrowsers.md` — Virtual browser definition in config.yml: embedded browser tabs for cloud console sign-in pages
- `cloud-accounts.md` — Cloud account configuration (AWS, Azure, GCP) in config.yml: dual-role IAM, learner vs admin permissions, env var injection
- `secrets.md` — Secrets block in config.yml: named secrets injected as environment variables, never committed to source
- `sandbox-presets.md` — Sandbox presets: pre-built environments replacing config.yml, no track_scripts directory needed
- `custom-resources.md` — Custom resources: Terraform modules provisioned as sandbox resources before other components
- `machine-types.md` — GCP machine types for VM sizing: machine_type field vs memory/cpus, available instance families and specs
- `quotas-and-limits.md` — Platform quotas and resource limits: sandbox host constraints, cloud project limits, script timeouts
- `variable-interpolation.md` — Variable interpolation: agent CLI variable setting in scripts, shortcode syntax in markdown
- `code-block-modifiers.md` — Code block modifiers: auto-run, copy button, line wrap, line numbers appended to fenced code blocks
- `buttons-and-navigation.md` — Inline buttons and navigation: tab switching, section navigation, external links, admonitions, collapsibles
- `ui-checks.md` — UI checks: virtual browser DOM element inspection and URL validation complementing lifecycle check scripts

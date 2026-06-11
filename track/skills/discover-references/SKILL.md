# Discover References

Find relevant track-format and error reference files for a given query.

## Progress reporting

Do not create your own top-level task list. The invoking command owns the user-facing task list. If this skill represents a single user-visible step in that list, your work maps to one entry. If you need finer-grained progress, update the existing task list rather than starting a new one.

## Paths

- Manifest: `${CLAUDE_PLUGIN_ROOT}/references/_index.md`

## Workflow

1. Read the manifest file
2. Match the query against file descriptions — use keyword overlap, synonyms, and related concepts
3. Return matching file paths with their one-line descriptions
4. The invoking agent reads the files it needs

## Scope

This skill covers:
- Track format references (`references/track-format/`)
- Infrastructure (`references/infrastructure/`)
- Decision frameworks (`references/decision-frameworks/`)
- Error references (`references/errors/`)
- Evaluation references (`references/evaluation/`)

For best practices and anti-patterns, use `discover-best-practices` instead.

## Usage

Agent invokes: "What references exist for service tab port configuration?"
Skill returns:
```
- track-format/tabs.md — Tab configuration: terminal, service, code, browser, website, external types
- track-format/config-yml.md — config.yml structure: sandbox infrastructure definition
```

Agent invokes: "How do I configure cloud accounts?"
Skill returns:
```
- track-format/cloud-accounts.md — Cloud account configuration (AWS, Azure, GCP)
- infrastructure/cloud-accounts/vm-plus-aws.md — VM + AWS account pattern
- infrastructure/cloud-accounts/vm-plus-azure.md — VM + Azure subscription pattern
- infrastructure/cloud-accounts/vm-plus-gcp.md — VM + GCP project pattern
- decision-frameworks/preset-vs-custom-config.md — When to use presets vs custom config.yml
```

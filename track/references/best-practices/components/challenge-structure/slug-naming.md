# Slug Naming

Evaluates whether challenge slugs are descriptive, consistent, and follow naming conventions that make the track readable as a table of contents.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Slugs are meaningless (challenge-1, lab-a, test), contain special characters, or use inconsistent casing |
| 2 | Below Standard | Slugs have some meaning but are inconsistent -- mixed naming styles, abbreviations without pattern, or overly long |
| 3 | Adequate | Slugs are lowercase and hyphenated, generally descriptive, but some are vague or do not match the challenge content |
| 4 | Good | Slugs are meaningful and consistent; lowercase, hyphenated; match directory names (minus number prefix) (production baseline) |
| 5 | Excellent | Slugs form a readable table of contents; someone can understand the track's flow just by reading the slug list |

## Guidance

Challenge slugs serve as both URL identifiers and human-readable labels in the codebase. They should be immediately meaningful to anyone reading the track source.

### Naming Rules

- **Lowercase only** -- no uppercase letters
- **Hyphen-separated** -- no underscores, spaces, or camelCase
- **Descriptive** -- the slug should hint at what the challenge covers
- **Concise** -- aim for 2-5 words; long enough to be meaningful, short enough to scan
- **ASCII only** -- no special characters, accents, or emoji

### Slug Must Match Directory Name

The slug in the challenge's assignment.md frontmatter must match the directory name (minus the number prefix):

```
Directory: 03-configure-vault-transit/
Slug:      configure-vault-transit
```

Good -- slugs form a readable table of contents:

```yaml
# Slug fields from each challenge's assignment.md frontmatter, shown together for comparison:
- slug: explore-the-cluster           # 01-explore-the-cluster/assignment.md
- slug: deploy-sample-application     # 02-deploy-sample-application/assignment.md
- slug: configure-ingress             # 03-configure-ingress/assignment.md
- slug: enable-monitoring             # 04-enable-monitoring/assignment.md
- slug: scale-under-load              # 05-scale-under-load/assignment.md
- slug: troubleshoot-connectivity     # 06-troubleshoot-connectivity/assignment.md
```

Reading just the slugs tells you: explore, deploy, configure networking, add monitoring, scale, then debug.

Good -- consistent verb-noun pattern:

```yaml
# Per-challenge assignment.md slugs, shown together for comparison:
- slug: create-vault-server           # 01-create-vault-server/assignment.md
- slug: configure-auth-method         # 02-configure-auth-method/assignment.md
- slug: write-secrets                 # 03-write-secrets/assignment.md
- slug: create-acl-policy             # 04-create-acl-policy/assignment.md
- slug: enable-audit-logging          # 05-enable-audit-logging/assignment.md
```

Bad -- meaningless slugs:

```yaml
# Per-challenge assignment.md slugs:
- slug: challenge-1
- slug: challenge-2
- slug: challenge-3
```

Bad -- inconsistent style:

```yaml
# Per-challenge assignment.md slugs:
- slug: create-vault-server       # verb-noun
- slug: auth_methods              # underscore, no verb
- slug: SecretManagement          # camelCase
- slug: step-4-do-the-thing       # number in slug, vague
```

Bad -- overly long:

```yaml
# Per-challenge assignment.md slug:
- slug: configure-the-hashicorp-vault-transit-secrets-engine-for-encryption-as-a-service
```

Bad -- overly abbreviated:

```yaml
# Per-challenge assignment.md slug:
- slug: cfg-vlt-trs
```

### Verb-First Convention

Starting slugs with an action verb creates consistency and signals what the learner will do:

| Verb | Use For |
|------|---------|
| explore- | Read-only observation challenges |
| create- | Building something new |
| configure- | Modifying settings or config |
| deploy- | Launching services or applications |
| verify- | Checking that something works |
| troubleshoot- | Diagnosing and fixing issues |
| cleanup- | Teardown and review |

### Slug vs Title

The slug is a URL-safe identifier, not the display title. The title (in the challenge's assignment.md frontmatter) can be more expressive:

```yaml
# In a challenge's assignment.md frontmatter:
slug: configure-ingress           # URL-safe, concise
title: Configure Ingress Routing  # Human-readable, can use capitals
```

Do not try to make the slug identical to the title -- they serve different purposes.

## What to Watch For

- Slugs that do not match their directory names (minus the number prefix)
- Mixed naming conventions within the same track (some verb-first, some noun-only, some abbreviated)
- Slugs containing numbers that duplicate the directory numbering (01-step-1-intro)
- Uppercase characters or underscores in slugs
- Slugs so vague that you cannot tell what the challenge is about (setup, next-steps, part-2)
- Slugs that duplicate across tracks in the same organization -- consider uniqueness for analytics

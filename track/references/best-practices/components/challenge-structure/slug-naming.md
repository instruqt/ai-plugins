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

The slug in track.yml must match the directory name (minus the number prefix):

```
Directory: 03-configure-vault-transit/
Slug:      configure-vault-transit
```

Good -- slugs form a readable table of contents:

```yaml
challenges:
- slug: explore-the-cluster
- slug: deploy-sample-application
- slug: configure-ingress
- slug: enable-monitoring
- slug: scale-under-load
- slug: troubleshoot-connectivity
```

Reading just the slugs tells you: explore, deploy, configure networking, add monitoring, scale, then debug.

Good -- consistent verb-noun pattern:

```yaml
challenges:
- slug: create-vault-server
- slug: configure-auth-method
- slug: write-secrets
- slug: create-acl-policy
- slug: enable-audit-logging
```

Bad -- meaningless slugs:

```yaml
challenges:
- slug: challenge-1
- slug: challenge-2
- slug: challenge-3
```

Bad -- inconsistent style:

```yaml
challenges:
- slug: create-vault-server       # verb-noun
- slug: auth_methods              # underscore, no verb
- slug: SecretManagement          # camelCase
- slug: step-4-do-the-thing       # number in slug, vague
```

Bad -- overly long:

```yaml
challenges:
- slug: configure-the-hashicorp-vault-transit-secrets-engine-for-encryption-as-a-service
```

Bad -- overly abbreviated:

```yaml
challenges:
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

The slug is a URL-safe identifier, not the display title. The title (in track.yml) can be more expressive:

```yaml
- slug: configure-ingress           # URL-safe, concise
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

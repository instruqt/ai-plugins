# Metadata Quality

Evaluates whether the track's title, teaser, and description are clear, accurate, and optimized for catalog discovery.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Title is vague or generic ("Lab 1"), teaser is missing, description is empty or a copy of the title |
| 2 | Below Standard | Title names the product but not the skill; teaser exists but is bland; description is a single sentence with no useful detail |
| 3 | Adequate | Title is descriptive, teaser is present and accurate, description covers the track's scope but lacks structure or branding |
| 4 | Good | Clear, accurate metadata; title is searchable; teaser is a compelling one-liner; description explains what the learner will do and learn (production baseline) |
| 5 | Excellent | Metadata optimized for catalog discovery -- title includes key search terms, teaser hooks the reader, description uses inline HTML for branding and structure |

## Guidance

Track metadata is the first thing a learner sees in the catalog. It determines whether they click into the track or scroll past it.

### Title

The title should be specific, searchable, and concise. It should tell the learner what they will do, not just what product is involved.

Good -- specific and searchable:

```yaml
title: Deploy a Kubernetes Application with Helm Charts
```

Good -- product + skill:

```yaml
title: HashiCorp Vault - Dynamic Database Credentials
```

Bad -- vague:

```yaml
title: Getting Started
```

Bad -- too long:

```yaml
title: Learn How to Use HashiCorp Terraform to Deploy Infrastructure on AWS Using Best Practices for State Management
```

### Teaser

The teaser is a single line displayed in catalog cards. It should be a compelling hook -- what will the learner be able to do after completing this track?

Good -- outcome-focused:

```yaml
teaser: Automate TLS certificate rotation without downtime using Vault's PKI engine.
```

Bad -- restates the title:

```yaml
teaser: This track teaches you about Vault PKI.
```

Bad -- too long (gets truncated in the UI):

```yaml
teaser: In this comprehensive hands-on lab you will learn everything about deploying applications to Kubernetes clusters using Helm charts including templates, values files, and release management.
```

### Description

The description supports inline HTML and can include branding, logos, and structured content. It should explain the track's scope, prerequisites, and learning outcomes.

Good -- structured with HTML:

```yaml
description: |-
  <img src="https://storage.googleapis.com/instruqt-frontend/img/tracks/hashicorp/vault-logo.png" width="200">

  In this track you will learn how to:
  - Configure Vault's PKI secrets engine
  - Generate dynamic TLS certificates
  - Set up automatic certificate rotation

  **Prerequisites:** Basic familiarity with Vault and TLS concepts.
```

Good -- plain text when branding is not needed:

```yaml
description: |-
  Learn to deploy containerized applications to Kubernetes using Helm charts.
  You will create charts from scratch, manage values files, and perform
  rolling upgrades.
```

Bad -- empty or placeholder:

```yaml
description: TODO
```

Bad -- copy of the teaser:

```yaml
description: Automate TLS certificate rotation without downtime using Vault's PKI engine.
```

## What to Watch For

- Titles that only name the product without specifying the skill or task
- Teasers longer than one line -- they get truncated in catalog card views
- Descriptions that are empty, placeholder text, or exact duplicates of the teaser
- Missing branding (logos, colors) when the track is part of a branded catalog
- HTML in descriptions that is broken or uses unsupported tags -- test rendering in the catalog preview
- Titles with internal jargon or abbreviations that external learners would not search for

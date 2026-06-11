# Pre-Challenge Notes Slide Design

Evaluates whether pre-challenge notes slides are configured to provide learners with context and engagement during challenge loading.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No notes defined; learners see the default "please be patient" loading message on every challenge |
| 2 | Below Standard | Notes exist but are a wall of text or contain irrelevant content that does not prepare the learner for the challenge |
| 3 | Adequate | Notes provide some context for the upcoming challenge, but are either too sparse or too verbose; slide structure is flat |
| 4 | Good | Notes are present for challenges that need introduction; multi-slide arrays break content into digestible pieces (production baseline) |
| 5 | Excellent | Notes create anticipation and context-set without overwhelming -- splash images build brand presence, text is concise and motivating, pacing matches loading time |

## Guidance

Notes slides appear during challenge loading, keeping learners engaged while the sandbox environment starts. They are defined in the challenge configuration as a `notes` array.

The `enhanced_loading` setting controls the loading experience:

- **Track level**: set `enhanced_loading: false` to use the notes-only loading screen (default behavior shows notes during load)
- **First challenge**: use `enhanced_loading: null` to inherit the track-level setting
- **Subsequent challenges**: set `enhanced_loading: false` explicitly to ensure notes display during transitions

Notes support three content types:

**Text slide**:

```yaml
notes:
  - type: text
    contents: |
      # Welcome to Kubernetes Basics

      In this challenge, you will deploy your first pod
      and explore the cluster using kubectl.
```

**Image slide**:

```yaml
notes:
  - type: image
    url: https://cdn.example.com/challenge-intro.png
```

**Video slide**:

```yaml
notes:
  - type: video
    url: https://www.youtube.com/embed/dQw4w9WgXcQ
```

**Multi-slide array** for complex introductions:

```yaml
notes:
  - type: image
    url: https://cdn.example.com/splash.png
  - type: text
    contents: |
      # What You Will Learn

      - Deploy a containerized application
      - Configure service networking
      - Monitor pod health
  - type: text
    contents: |
      # Before You Begin

      This challenge builds on the previous one.
      Make sure your deployment from Challenge 1 is still running.
```

**Splash image + text intro pattern** -- the most common and effective pattern for first challenges:

```yaml
notes:
  - type: image
    url: https://cdn.example.com/product-logo-splash.png
  - type: text
    contents: |
      # Getting Started with Terraform

      Terraform is an infrastructure-as-code tool that lets you
      define cloud resources in declarative configuration files.

      In this track, you will provision your first infrastructure
      on AWS using Terraform.
```

Good -- multi-slide with image and focused text:

```yaml
notes:
  - type: image
    url: https://cdn.example.com/challenge-3-intro.png
  - type: text
    contents: |
      # Scaling Your Application

      Now that your app is running, it is time to handle
      real-world traffic by scaling to multiple replicas.
```

Good -- subsequent challenge with explicit enhanced_loading:

```yaml
enhanced_loading: false
notes:
  - type: text
    contents: |
      # Next Up: Monitoring

      You will configure Prometheus to collect metrics
      from your running application.
```

Bad -- no notes on a challenge with a 60-second setup time:

```yaml
# (no notes defined -- learner stares at "please be patient")
```

Bad -- single text slide with an essay:

```yaml
notes:
  - type: text
    contents: |
      # Introduction

      Kubernetes is an open-source container orchestration platform
      originally developed by Google based on their internal Borg
      system. It was open-sourced in 2014 and is now maintained by
      the Cloud Native Computing Foundation. Kubernetes automates
      deployment, scaling, and management of containerized applications.
      It groups containers that make up an application into logical
      units for easy management and discovery. Kubernetes builds upon
      15 years of experience of running production workloads at Google,
      combined with best-of-breed ideas and practices from the community.
      [... 500 more words ...]
```

Bad -- notes that duplicate the assignment content:

```yaml
notes:
  - type: text
    contents: |
      # Steps

      1. Run kubectl get pods
      2. Edit the deployment.yaml file
      3. Apply the changes
      # (this belongs in the assignment, not in notes)
```

## What to Watch For

- Notes are a loading screen, not a lesson -- keep them short and motivating, not instructional
- The first challenge benefits most from a splash image + brief introduction; subsequent challenges need lighter context-setting
- `enhanced_loading: false` must be set at the track level and/or per challenge for the notes-only loading experience to work
- First challenge should use `enhanced_loading: null` to inherit the track setting; subsequent challenges should set `enhanced_loading: false` explicitly
- Multi-slide arrays auto-advance, so each slide should be readable in a few seconds
- Do not duplicate assignment content in notes -- notes set context, assignments give instructions
- Video slides should be short (30-60 seconds); learners cannot pause them during loading
- Image URLs must be publicly accessible HTTPS URLs; broken images show as empty slides

# Instruqt Plugin Marketplace

A collection of Claude Code plugins for building and managing Instruqt content.

## Available Plugins

| Plugin | Description |
|--------|-------------|
| **track** | End-to-end track generation — from customer research through challenge implementation with built-in quality scoring |

## Setup

### 1. Add the marketplace

```
/plugin marketplace add instruqt/ai-plugins
```

### 2. Install a plugin

```
/plugin install track@instruqt
```

Run `/reload-plugins` to activate.

## Track Plugin

The track plugin provides a guided pipeline for creating Instruqt tracks. Each stage builds on the previous one, producing structured documents that feed into later stages.

### Workflow

#### Research

Start by building customer context. This scrapes the company website and produces a company profile and style guide.

```
/track:research-company url:https://example.com
```

Then research the specific product the track will cover. Repeat for each product if needed.

```
/track:research-product "Product Name"
```

#### Plan

Create the track plan — this is an interactive conversation where the planner asks about your intent, target audience, and learning objectives, then produces a challenge roadmap.

```
/track:plan-track <topic>
```

Optionally review the track plan for quality:

```
/track:review-track-plan
```

Then plan each challenge in detail. Challenges must be planned in order since each one builds on the previous.

```
/track:plan-challenge <challenge-slug>
```

Optionally review individual challenge plans:

```
/track:review-challenge-plan <challenge-slug>
```

#### Generate

Generate challenges one at a time:

```
/track:generate-challenge <challenge-slug>
```

Or generate the entire track at once — this picks up where you left off and generates all remaining challenges in sequence:

```
/track:generate-track
```

After each challenge is generated, you can choose to:
- **Test automatically** — runs `instruqt track test`
- **Test manually** — you test, then confirm when done
- **Skip testing** — continue to the next challenge

#### Finish up

Generate a README for track authors and maintainers:

```
/track:generate-readme
```

Review the fully generated track against quality rubrics:

```
/track:review-track
```

### Command Reference

| Command | Description | Prerequisites |
|---------|-------------|---------------|
| `research-company` | Build customer profile and style guide from company website | None |
| `research-product` | Research a specific product in depth | Company research |
| `plan-track` | Define track scope, audience, objectives, and challenge roadmap | Company research |
| `plan-challenge` | Create detailed plan for a single challenge | Track plan |
| `generate-challenge` | Implement a single challenge (content + scripts) | Challenge plan |
| `generate-track` | Generate all remaining challenges in sequence | Track plan + challenge plans |
| `generate-readme` | Produce a maintainer-facing README | At least one generated challenge |
| `review-track-plan` | Score the track plan against quality rubrics | Track plan |
| `review-challenge-plan` | Score a challenge plan against quality rubrics | Challenge plan |
| `review-track` | Score the full generated track against quality rubrics | Generated track |

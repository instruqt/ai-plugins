# Instruqt Plugin Marketplace

A collection of Claude Code plugins for building and managing Instruqt content.

## Available Plugins

| Plugin | Description |
|--------|-------------|
| **track** | End-to-end track generation — from company and product research through challenge implementation with built-in quality scoring |

## Prerequisites

**Claude Code** — the plugin runs inside Claude Code (CLI, desktop app, web app, or the VS Code / JetBrains extensions).

- **Linux** and **macOS**: supported natively
- **Windows**: supported via [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install) (use the CLI or VS Code with the WSL remote extension)
- **Web** (claude.ai/code): supported — runs on a cloud Linux VM

**Recommended (optional):**

- **[Instruqt CLI](https://docs.instruqt.com/docs/cli-installation)** — enables `instruqt track validate` and `instruqt track test` for server-side validation and end-to-end testing. Without it, the plugin still runs its own structural checks, but can't catch platform-specific issues like slug conflicts or version requirements.
- **[shellcheck](https://github.com/koalaman/shellcheck)** — enables lint-quality checks on lifecycle scripts. Without it, script validation is less thorough.

## Setup

### 1. Add the marketplace

```
/plugin marketplace add instruqt/ai-plugins
```

### 2. Install a plugin

```
/plugin install track@ai-plugin
```

Run `/reload-plugins` to activate.

## Track Plugin

The track plugin walks you through the full process of creating an Instruqt track — from researching your company and products, through planning, all the way to generating working challenges. Each stage builds on the previous one, so the content stays consistent and technically accurate throughout.

The plugin has a built-in knowledge base of Instruqt best practices — covering everything from script structure and infrastructure patterns to content design and educational flow. Plans and generated content follow these best practices by default. At each stage you can also run a review, which scores your output against the same rubrics to catch anything that slipped through — but it's an extra safeguard, not the only line of defense.

### Workflow

The plugin provides a staged workflow that guides you through the full content creation process — from research through planning to generation. Each stage feeds into the next, so you always have the right context at the right time.

> [!TIP]
> **Clear your context between stages.** Run `/clear` before starting each new stage (Research → Plan → Generate). Each stage writes its output to specific files — company profiles, product profiles, track plans, challenge plans — and the next stage reads those files as its starting context. This means you don't need to carry conversation history forward. Clearing between stages:
> - **Keeps the context clean** — no stale or irrelevant information bleeding in from earlier stages.
> - **Gives you full control over compaction** — you decide what context carries forward (via the files each stage produces), rather than relying on the LLM's automatic summarization to guess what to keep and what to drop.

#### Research

The plugin scrapes your company website to understand your branding, messaging, terminology, and tone — then distills it into a company profile and writing style guide. Everything generated later uses this context, so your tracks read like *you* wrote them, not a generic AI. You only need to do this once per company — the profile is reused across all your tracks.

```
/track:research-company <url>

e.g.
/track:research-company https://norncorp.com
```

Next, research the specific product your track will cover. This captures your product's capabilities, architecture, key concepts, and documentation. The more the plugin knows about your product, the more technically accurate your challenges will be — no hallucinated features or made-up workflows. Like company research, you only need to do this once per product — the profile carries over to any track that covers it.

```
/track:research-product <product-name>

e.g.
/track:research-product Mimir
```

> [!NOTE]
> Research artifacts are stored in `~/.instruqt/` by default (override with `INSTRUQT_DATA_DIR`). Company profiles go in `companies/<slug>/` and product profiles in `products/<company-slug>/<product-slug>/`. These are plain files, so you can check them into version control or share them with teammates.

#### Plan

Now you'll define what your track teaches and who it's for. The planner has an interactive conversation with you to nail down your target audience, learning objectives, and an outline of the challenges included. Spending time here pays off — a clear plan means every challenge has a purpose in the learning arc, and you avoid rework later.

```
/track:plan-track <description>

e.g.
/track:plan-track A hands-on intro to Mimir secrets management for platform engineers
```

Once the plan is generated, read through it carefully. This is the foundation everything else builds on — challenge plans, generated content, and scripts all flow from what's defined here. If something feels off — wrong audience level, missing objectives, challenges that don't quite fit — now is the time to iterate. Talk to the AI, ask it to adjust, and keep refining until the plan reflects what you actually want to build.

You can also run a structured review against quality rubrics to catch issues you might miss on a read-through, like objectives that don't map to challenges or an audience definition that's too vague:

```
/track:review-track-plan
```

Then plan each challenge in detail. Pick a challenge slug from the outline in your track plan — that's where the slugs are defined. This is where you specify what happens step by step: the assignment outline, tab layout, infrastructure changes, and lifecycle scripts (setup, check, solve). You're designing the learning experience — what your learners read, what they do, and how the platform validates their work. Plan challenges in order, since each one builds on what came before.

```
/track:plan-challenge <challenge-slug>
```

You can review individual challenge plans too, to catch issues like unrealistic time estimates, missing check logic, or infrastructure gaps:

```
/track:review-challenge-plan <challenge-slug>
```

#### Generate

This is where your plans become a working track. The generator produces everything that makes up a challenge — the assignment markdown, tab configuration, sandbox resources, setup scripts that provision the environment, check scripts that validate your learner's work, and solve scripts that demonstrate the solution. It runs internal quality scoring and validation (shellcheck, `instruqt track validate`) automatically, so you catch errors before you ever push.

We recommend generating challenges one at a time. It's easier to review each challenge as the author, the AI works with less context so output stays more consistent, and you can test each challenge before moving on — making sure the foundation is solid before building on top of it.

```
/track:generate-challenge <challenge-slug>
```

You could also choose to do all challenges at once (even if you have already generated individual challenges before). This picks up where you left off and runs through them in sequence, but you lose the ability to catch issues early:

```
/track:generate-all-challenges
```

After each challenge is generated, you choose what to do next:
- **Test automatically** — runs `instruqt track test`
- **Test manually** — you test it yourself, then confirm when done
- **Skip testing** — move on to the next challenge

You can also review a single generated challenge against quality rubrics before moving on:

```
/track:review-challenge <challenge-slug>
```

#### Finish up

Generate a README that documents your track's sandbox architecture, host and tab layout, and challenge map. Handy for anyone on your team who needs to maintain or extend the track later.

```
/track:generate-readme
```

Finally, review the fully generated track against quality rubrics. This is a holistic pass across all your challenges, checking content coherence, script correctness, educational progression, and brand consistency:

```
/track:review-track
```

### Command Reference

| Command | Description | Prerequisites |
|---------|-------------|---------------|
| `research-company` | Build company profile and style guide from company website | None |
| `research-product` | Research a specific product in depth | None |
| `plan-track` | Define track scope, audience, objectives, and an outline of the challenges included | None (company/product context improves quality) |
| `plan-challenge` | Create detailed plan for a single challenge | Track plan |
| `generate-challenge` | Implement a single challenge (content + scripts) | Challenge plan |
| `generate-all-challenges` | Generate all remaining challenges in sequence | Track plan + challenge plans |
| `generate-readme` | Produce a maintainer-facing README | At least one generated challenge |
| `review-track-plan` | Score the track plan against quality rubrics | Track plan |
| `review-challenge-plan` | Score a challenge plan against quality rubrics | Challenge plan |
| `review-challenge` | Score a single generated challenge against quality rubrics | Generated challenge |
| `review-track` | Score the full generated track against quality rubrics | Generated track |

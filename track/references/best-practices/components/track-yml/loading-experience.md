# Loading Experience

Evaluates whether the track's loading experience -- enhanced_loading, notes slides, and loadingMessages -- is configured to engage learners during sandbox provisioning rather than showing a blank wait screen.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No loading experience configured; learner stares at a spinner with no context while the sandbox provisions |
| 2 | Below Standard | enhanced_loading or notes configured but not correctly -- notes do not display, or loading messages conflict with notes slides |
| 3 | Adequate | Loading messages enabled OR notes slides present, but not both working together; transitions between loading and first challenge are abrupt |
| 4 | Good | Notes display correctly during provisioning; enhanced_loading settings are correct per challenge; loadingMessages do not conflict (production baseline) |
| 5 | Excellent | Loading experience creates anticipation -- splash image followed by text intro via notes, custom loading messages match brand, smooth transition into first challenge |

## Guidance

The loading experience spans three mechanisms that must work together: `enhanced_loading` (in track.yml), notes slides (in each challenge's assignment.md frontmatter), and `loadingMessages` (in track.yml's lab_config).

**Important:** `notes`, `tabs`, and `enhanced_loading` at the challenge level are defined in each challenge's `assignment.md` YAML frontmatter -- NOT in `track.yml`. The `track.yml` file only contains track-level fields like `enhanced_loading`, `lab_config`, and track metadata. There is no `challenges:` array in `track.yml`.

### enhanced_loading

This field controls the initial loading experience. It exists at two levels:

- **`enhanced_loading: true`** -- drops the learner directly into the lab immediately. The sandbox provisions in the background while the learner can already see and interact with the environment. No notes slides are shown.
- **`enhanced_loading: false`** (or omitted) -- shows a loading screen during sandbox provisioning. Notes slides display during this time, keeping the learner engaged while they wait.

**Best practice:** Use `enhanced_loading: true` when sandbox startup is fast (< 15 seconds). Use `enhanced_loading: false` with notes slides when startup is slow -- the notes keep the learner engaged during the wait.

**Track level** -- set in `track.yml`:

```yaml
# track.yml -- fast startup, drop learner straight into the lab
enhanced_loading: true
```

```yaml
# track.yml -- slow startup, show notes slides while provisioning
enhanced_loading: false
```

**Per-challenge level** -- set in the challenge's `assignment.md` frontmatter to control transition behavior:

- **First challenge**: leave as `null` (omit the field) or set to `false`. The first challenge uses the track-level setting.
- **Subsequent challenges**: set to `false` to allow notes slides to display during challenge transitions.

Good -- slow-starting track uses notes to engage learners during provisioning:

```yaml
# track.yml
enhanced_loading: false
```

```yaml
# 01-intro/assignment.md (YAML frontmatter)
slug: intro
type: challenge
title: Introduction
notes:
- type: image
  url: https://storage.googleapis.com/instruqt/splash/vault-intro.png
- type: text
  contents: |-
    # Welcome to Vault Transit Encryption
    In this track you will learn to encrypt application data...
tabs:
- title: Terminal
  type: terminal
  hostname: workstation
```

Good -- fast-starting track drops learner straight into the lab:

```yaml
# track.yml
enhanced_loading: true
```

Good -- subsequent challenge assignment.md frontmatter with explicit enhanced_loading override:

```yaml
# 02-configure-transit/assignment.md (YAML frontmatter)
slug: configure-transit
type: challenge
title: Configure Transit Engine
enhanced_loading: false
notes:
- type: text
  contents: |-
    # Configuring the Transit Engine
    Next you will enable and configure Vault's transit secrets engine...
tabs:
- title: Terminal
  type: terminal
  hostname: workstation
```

### Notes Slides

Notes are per-challenge content defined in the challenge's `assignment.md` frontmatter. They are shown during sandbox provisioning or challenge transitions. They support two types:

- **image** -- a splash image (URL to PNG/JPG). Displayed full-screen during loading.
- **text** -- markdown-formatted text. Displayed as a readable overlay.

The recommended pattern for the first challenge is: image slide first (splash/branding), then text slide (introduction).

Good -- splash image followed by text intro in assignment.md frontmatter:

```yaml
# 01-intro/assignment.md (YAML frontmatter)
slug: intro
type: challenge
title: Introduction
notes:
- type: image
  url: https://storage.googleapis.com/instruqt/splash/product-splash.png
- type: text
  contents: |-
    # Welcome
    This track walks you through deploying your first application.

    **What you'll need:**
    - Basic terminal familiarity
    - No prior cloud experience required
tabs:
- title: Terminal
  type: terminal
  hostname: workstation
```

Good -- subsequent challenge with context-setting note:

```yaml
# 02-deploy-app/assignment.md (YAML frontmatter)
slug: deploy-app
type: challenge
title: Deploy the Application
enhanced_loading: false
notes:
- type: text
  contents: |-
    # Deploying the Application
    Now that your cluster is running, you will deploy a sample
    application and expose it via a load balancer.
tabs:
- title: Terminal
  type: terminal
  hostname: workstation
```

Bad -- notes defined but enhanced_loading: true skips them (learner goes straight to lab, never sees notes):

```yaml
# track.yml -- immediate lab start, notes will not display
enhanced_loading: true
```

```yaml
# 01-intro/assignment.md (YAML frontmatter) -- these notes are wasted
slug: intro
type: challenge
title: Introduction
notes:
- type: text
  contents: "Welcome!"
```

### loadingMessages and Notes Interaction

`loadingMessages` in track.yml's `lab_config` and notes slides in assignment.md frontmatter serve similar purposes. When both are active, they can compete for attention. Choose one primary mechanism:

- **Notes slides** for rich, branded, multi-slide loading experiences
- **loadingMessages** for simple text messages during provisioning

Good -- notes as primary, loading messages disabled in track.yml:

```yaml
# track.yml
enhanced_loading: false
lab_config:
  loadingMessages: false
```

```yaml
# 01-intro/assignment.md (YAML frontmatter) -- notes defined per challenge
slug: intro
type: challenge
title: Introduction
notes:
- type: image
  url: https://storage.googleapis.com/instruqt/splash/product-splash.png
- type: text
  contents: |-
    # Welcome
    Your environment is being prepared...
```

Good -- loading messages as primary in track.yml, no notes in assignment.md:

```yaml
# track.yml
lab_config:
  loadingMessages:
  - "Provisioning your environment..."
  - "Installing dependencies..."
  - "Almost ready..."
```

Good -- branded custom loading messages that match the track's topic:

```yaml
# track.yml
lab_config:
  loadingMessages:
  - "Spinning up your Kubernetes cluster..."
  - "Deploying sample microservices..."
  - "Configuring monitoring stack..."
  - "Your environment is almost ready!"
```

Bad -- both active and conflicting:

```yaml
# track.yml
lab_config:
  loadingMessages: true
```

```yaml
# 01-intro/assignment.md (YAML frontmatter) -- competes with loading messages
slug: intro
type: challenge
title: Introduction
notes:
- type: text
  contents: "Welcome!"
```

## What to Watch For

- `enhanced_loading: true` means the learner goes straight into the lab -- notes slides will not display. Only use `true` when startup is fast enough that notes aren't needed.
- Notes slides defined in assignment.md but `enhanced_loading: true` at track level -- the notes are wasted, learners never see them
- loadingMessages and notes slides both active, creating a cluttered loading screen
- First challenge missing a notes slide when startup is slow -- the initial provisioning (longest wait) has no content
- Image URLs in notes that are broken or slow to load -- test them before publishing
- Notes text that is too long -- learners skim during loading, keep it concise and scannable
- Subsequent challenges missing enhanced_loading: false in their assignment.md when they have notes -- notes will not display on transition
- Custom `loadingMessages` arrays are a lightweight alternative to notes when you want branded messages without the overhead of splash images

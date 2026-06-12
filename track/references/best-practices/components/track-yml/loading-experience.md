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

This field controls whether the Instruqt platform shows the enhanced loading overlay. It exists at two levels:

**Track level** -- set to `false` in `track.yml` to allow notes slides to display as the loading overlay:

```yaml
# track.yml (top level)
enhanced_loading: false
```

**Per-challenge level** -- set in the challenge's `assignment.md` frontmatter to control transition behavior:

- **First challenge**: leave as `null` (omit the field) or set to `false`. The first challenge uses the track-level setting.
- **Subsequent challenges**: set to `false` to allow notes slides to display during challenge transitions.

Good -- track.yml sets enhanced_loading at track level:

```yaml
# track.yml
enhanced_loading: false
```

Good -- first challenge assignment.md frontmatter with notes (inherits track-level enhanced_loading):

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

Bad -- enhanced_loading: true at track level blocks notes slides:

```yaml
# track.yml -- notes slides will not display during loading
enhanced_loading: true
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

Bad -- notes defined in assignment.md but track-level enhanced_loading not set to false:

```yaml
# track.yml -- this blocks notes slides from displaying
enhanced_loading: true
```

```yaml
# 01-intro/assignment.md (YAML frontmatter) -- these notes will not display
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

- Track-level enhanced_loading not set to false in track.yml when notes slides are defined in assignment.md -- notes will not display
- Notes slides defined in assignment.md but never seen because enhanced_loading overrides them
- loadingMessages and notes slides both active, creating a cluttered loading screen
- First challenge missing a notes slide -- the initial provisioning (longest wait) has no content
- Image URLs in notes that are broken or slow to load -- test them before publishing
- Notes text that is too long -- learners skim during loading, keep it concise and scannable
- Subsequent challenges missing enhanced_loading: false in their assignment.md when they have notes -- notes will not display on transition

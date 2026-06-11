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

The loading experience spans three mechanisms that must work together: `enhanced_loading` (track-level and per-challenge), notes slides (per-challenge), and `loadingMessages` (in lab_config).

### enhanced_loading

This field controls whether the Instruqt platform shows the enhanced loading overlay. It exists at two levels:

**Track level** -- set to `false` to allow notes slides to display as the loading overlay:

```yaml
# track.yml (top level)
enhanced_loading: false
```

**Per-challenge level** -- controls the transition behavior for each challenge:

- **First challenge**: leave as `null` (omit the field) or set to `false`. The first challenge uses the track-level setting.
- **Subsequent challenges**: set to `false` to allow notes slides to display during challenge transitions.

Good -- track-level setting with per-challenge overrides:

```yaml
# track.yml (top level)
enhanced_loading: false

challenges:
- slug: intro
  # enhanced_loading not set (null) -- inherits track-level false
  notes:
  - type: image
    url: https://storage.googleapis.com/instruqt/splash/vault-intro.png
  - type: text
    contents: |-
      # Welcome to Vault Transit Encryption
      In this track you will learn to encrypt application data...

- slug: configure-transit
  enhanced_loading: false
  notes:
  - type: text
    contents: |-
      # Configuring the Transit Engine
      Next you will enable and configure Vault's transit secrets engine...
```

Bad -- enhanced_loading: true at track level blocks notes slides:

```yaml
# Notes slides will not display during loading
enhanced_loading: true
```

### Notes Slides

Notes are per-challenge content shown during sandbox provisioning or challenge transitions. They support two types:

- **image** -- a splash image (URL to PNG/JPG). Displayed full-screen during loading.
- **text** -- markdown-formatted text. Displayed as a readable overlay.

The recommended pattern for the first challenge is: image slide first (splash/branding), then text slide (introduction).

Good -- splash image followed by text intro:

```yaml
challenges:
- slug: intro
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
```

Good -- subsequent challenge with context-setting note:

```yaml
- slug: deploy-app
  enhanced_loading: false
  notes:
  - type: text
    contents: |-
      # Deploying the Application
      Now that your cluster is running, you will deploy a sample
      application and expose it via a load balancer.
```

Bad -- notes defined but enhanced_loading not set to false:

```yaml
# These notes will not display
enhanced_loading: true
challenges:
- slug: intro
  notes:
  - type: text
    contents: "Welcome!"
```

### loadingMessages and Notes Interaction

`loadingMessages` in `lab_config` and notes slides serve similar purposes. When both are active, they can compete for attention. Choose one primary mechanism:

- **Notes slides** for rich, branded, multi-slide loading experiences
- **loadingMessages** for simple text messages during provisioning

Good -- notes as primary, loading messages disabled:

```yaml
lab_config:
  loadingMessages: false

# enhanced_loading: false at track level
# Notes slides defined per challenge
```

Good -- loading messages as primary, no notes:

```yaml
lab_config:
  loadingMessages:
  - "Provisioning your environment..."
  - "Installing dependencies..."
  - "Almost ready..."

# No notes slides defined
```

Bad -- both active and conflicting:

```yaml
lab_config:
  loadingMessages: true  # Default messages shown

challenges:
- slug: intro
  notes:
  - type: text
    contents: "Welcome!"  # Competes with loading messages
```

## What to Watch For

- Track-level enhanced_loading not set to false when notes slides are defined -- notes will not display
- Notes slides defined but never seen because enhanced_loading overrides them
- loadingMessages and notes slides both active, creating a cluttered loading screen
- First challenge missing a notes slide -- the initial provisioning (longest wait) has no content
- Image URLs in notes that are broken or slow to load -- test them before publishing
- Notes text that is too long -- learners skim during loading, keep it concise and scannable
- Subsequent challenges missing enhanced_loading: false when they have notes -- notes will not display on transition

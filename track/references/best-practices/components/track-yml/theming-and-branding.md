# Theming and Branding

Evaluates whether the track's visual identity -- theme, icon, description HTML, and loading messages -- creates a cohesive, branded experience.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No theming applied; default icon; description is plain text with no branding; track looks generic |
| 2 | Below Standard | Theme selected but inconsistent with other tracks in the catalog; icon present but low quality or mismatched; description HTML is broken |
| 3 | Adequate | Theme and icon are reasonable, description includes some structure, but the overall experience does not feel intentionally branded |
| 4 | Good | Consistent theming across the track; icon matches the product or topic; description uses HTML for structure (production baseline) |
| 5 | Excellent | Fully branded experience matching customer style -- custom theme, branded icon, HTML description with logo and colors, custom loading messages |

## Guidance

### Theme Selection

The `theme.name` field in `lab_config` controls the overall UI appearance:

- **modern-dark** -- Dark background, high contrast. Recommended default for most tracks. Reduces eye strain for terminal-heavy work.
- **original** -- Light theme. Use when the customer's brand guidelines require a light UI or when screenshots in the assignment were taken against a light background.

```yaml
lab_config:
  theme:
    name: modern-dark
```

### Icon

The track icon appears in catalog listings and the track header. Three formats are supported:

- **Path** -- relative path to an image file in the track repository
- **URL** -- absolute URL to a hosted image (PNG, SVG preferred)
- **Emoji** -- a single emoji character (fallback option, not recommended for branded tracks)

Good -- hosted logo URL:

```yaml
icon: https://storage.googleapis.com/instruqt-frontend/img/tracks/hashicorp/vault.png
```

Good -- local path:

```yaml
icon: assets/track-icon.png
```

Acceptable -- emoji fallback:

```yaml
icon: "\U0001F512"
```

Bad -- broken URL or placeholder:

```yaml
icon: https://example.com/TODO.png
```

### Description HTML

The description field supports inline HTML for rich formatting. Use it to include logos, structured lists, and visual hierarchy.

Good -- branded description:

```yaml
description: |-
  <img src="https://storage.googleapis.com/instruqt-frontend/img/tracks/hashicorp/vault-logo.png" width="200" alt="Vault Logo">

  <h3>What You'll Learn</h3>
  <ul>
    <li>Configure Vault's transit secrets engine</li>
    <li>Encrypt and decrypt application data</li>
    <li>Implement key rotation policies</li>
  </ul>

  <strong>Duration:</strong> ~45 minutes<br>
  <strong>Difficulty:</strong> Intermediate
```

Bad -- unstyled wall of text:

```yaml
description: This track teaches you about Vault transit. You will learn to encrypt data and rotate keys. It takes about 45 minutes.
```

### Loading Messages

Custom loading messages reinforce branding and set expectations during sandbox provisioning. They replace the default generic messages.

Good -- branded loading messages:

```yaml
lab_config:
  loadingMessages:
  - "Spinning up your HashiCorp Vault cluster..."
  - "Configuring authentication backends..."
  - "Seeding demo secrets and policies..."
  - "Almost ready -- finalizing your environment..."
```

Good -- disabled in favor of notes slides:

```yaml
lab_config:
  loadingMessages: false
```

### Consistency Across a Catalog

When building multiple tracks for the same customer or product, ensure:

- Same theme across all tracks
- Same icon style (all logos, or all from the same icon set)
- Same description HTML structure
- Same loading message tone and style

## What to Watch For

- Mixed themes within a catalog (some dark, some light) without deliberate reason
- Broken image URLs in icon or description fields -- test them in a browser
- Description HTML that renders differently than intended -- check for unclosed tags
- Loading messages that are too technical or reveal internal infrastructure details
- Emoji icons on tracks that are part of a branded catalog -- they look unprofessional alongside logo-based icons
- Missing alt text on description images -- accessibility concern

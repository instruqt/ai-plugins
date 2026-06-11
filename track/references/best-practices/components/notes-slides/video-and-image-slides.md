# Video and Image Slide Content

Evaluates whether video and image slides use correct formats, appropriate sizing, and add meaningful visual context to the learning experience.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Media fails to load -- broken URLs, wrong embed format, or unsupported file types |
| 2 | Below Standard | Media loads but is poorly sized (stretched, cropped, or tiny), or content is irrelevant to the challenge |
| 3 | Adequate | Media loads correctly and is relevant, but no attention to sizing, clipping, or visual polish |
| 4 | Good | Media loads correctly and is appropriately sized; videos are concise; images are clear and relevant (production baseline) |
| 5 | Excellent | Media is concise, branded, and adds context that text alone cannot -- videos are clipped to the relevant segment, images use inline CSS for polished presentation |

## Guidance

Notes slides support video and image content types for visual introductions during challenge loading.

### Video slides

Use `type: video` with a YouTube embed URL or a direct .mp4 URL:

```yaml
notes:
  - type: video
    url: https://www.youtube.com/embed/dQw4w9WgXcQ
```

YouTube embed URLs support start and end clipping parameters to show only the relevant segment:

```yaml
notes:
  - type: video
    url: https://www.youtube.com/embed/dQw4w9WgXcQ?start=30&end=90
```

Direct .mp4 files work for custom or pre-recorded content:

```yaml
notes:
  - type: video
    url: https://cdn.example.com/intro-challenge-3.mp4
```

### Image slides

Use `type: image` with an HTTPS URL:

```yaml
notes:
  - type: image
    url: https://cdn.example.com/architecture-diagram.png
```

### Inline CSS for styling

Notes slides support inline `<style>` tags within text slides for per-slide CSS customization. This is useful for controlling image presentation when combining text and images in a single text slide:

```yaml
notes:
  - type: text
    contents: |
      <style>
      img {
        display: block;
        margin: 0 auto;
        max-width: 80%;
      }
      </style>

      # Architecture Overview

      ![Architecture](https://cdn.example.com/arch-diagram.png)

      The diagram above shows how the components connect.
```

Centering and constraining image width:

```yaml
notes:
  - type: text
    contents: |
      <style>
      img { display: block; margin: 0 auto; max-width: 600px; }
      h1 { text-align: center; }
      </style>

      # Welcome

      ![Logo](https://cdn.example.com/company-logo.png)
```

Good -- YouTube video clipped to relevant segment:

```yaml
notes:
  - type: video
    url: https://www.youtube.com/embed/abc123?start=15&end=75
```

Good -- image with text context in a multi-slide sequence:

```yaml
notes:
  - type: image
    url: https://cdn.example.com/product-splash.png
  - type: text
    contents: |
      # What You Will Build

      By the end of this challenge, your architecture
      will look like the diagram on the previous slide.
```

Good -- inline CSS for polished image presentation:

```yaml
notes:
  - type: text
    contents: |
      <style>
      img { max-width: 500px; display: block; margin: 20px auto; }
      </style>

      # Network Topology

      ![Topology](https://cdn.example.com/topology.png)

      Three subnets connected through a transit gateway.
```

Bad -- YouTube watch URL instead of embed URL:

```yaml
notes:
  - type: video
    url: https://www.youtube.com/watch?v=dQw4w9WgXcQ
```

Bad -- HTTP image URL (will be blocked):

```yaml
notes:
  - type: image
    url: http://example.com/diagram.png
```

Bad -- 10-minute video for a loading screen:

```yaml
notes:
  - type: video
    url: https://www.youtube.com/embed/abc123
    # Full 10-minute product demo -- learner cannot pause or skip
```

Bad -- massive unscaled image that overflows the slide:

```yaml
notes:
  - type: image
    url: https://cdn.example.com/screenshot-4000x3000.png
    # No sizing control, image renders at full resolution
```

## What to Watch For

- YouTube URLs must use the `/embed/` format, not `/watch?v=` -- the latter will not render in the notes player
- Use `?start=N&end=N` on YouTube embeds to clip to the relevant segment; learners cannot control playback during loading
- Keep videos under 60 seconds -- loading screens are brief and learners may advance past them
- .mp4 files must be hosted on HTTPS URLs accessible from the learner's browser (not from inside the sandbox)
- Image URLs must be HTTPS; HTTP URLs are blocked by mixed-content policies
- Large images should use inline CSS (`max-width`, `display: block`, `margin: auto`) to prevent overflow and ensure centering
- Inline `<style>` tags work in text-type slides, not in image-type or video-type slides -- combine markup in a text slide if you need styled images
- Branded splash images for the first challenge create a professional impression; use the customer's color scheme and logo where appropriate

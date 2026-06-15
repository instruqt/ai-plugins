# Lab Config

Evaluates whether lab_config sub-fields are configured to create an optimal layout and interaction experience for the track's content type.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No lab_config specified, relying entirely on platform defaults that may not suit the content |
| 2 | Below Standard | lab_config present but layout mismatched to content -- reading-heavy track with a tiny sidebar, or code-heavy track with an oversized sidebar |
| 3 | Adequate | Layout is functional but not optimized -- default sidebar size when the content would benefit from adjustment, or missing feedback settings |
| 4 | Good | Layout matches content type; sidebar_size appropriate; feedback settings configured; default_layout chosen deliberately (production baseline) |
| 5 | Excellent | `loadingMessages` disabled with notes slides handling loading engagement; layout tested across content types; sidebar and layout create an optimal reading-to-doing ratio |

## Guidance

The `lab_config` block in track.yml controls the learner's UI experience. Key sub-fields:

### default_layout

The layout determines how the assignment panel and sandbox tabs are arranged.

- **AssignmentRight** -- Assignment text on the right, sandbox on the left. Use for ~80% of tracks. Works best when learners need to read instructions while interacting with a terminal or editor.
- **AssignmentLeft** -- Assignment on the left, sandbox on the right. Rarely used; consider only when the primary tab is a wide dashboard that benefits from right-side placement.
- **AssignmentBottom** -- Assignment at the bottom, sandbox fills the top. Good for tracks where the sandbox (a web UI, dashboard, or wide terminal) needs maximum horizontal space.

Good -- standard layout for most tracks:

```yaml
lab_config:
  default_layout: AssignmentRight
```

### sidebar_size

Controls the width percentage of the assignment panel. The right value depends on content density:

- **30** -- Code-heavy tracks where the learner spends most time in the terminal. Keeps the sandbox area large.
- **33** -- Default balance. Works for most tracks with moderate instruction length.
- **40-45** -- Reading-heavy tracks with detailed explanations, diagrams referenced in text, or long-form instructions.

Good -- code-heavy track with narrow sidebar:

```yaml
lab_config:
  default_layout: AssignmentRight
  sidebar_size: 30
```

Good -- reading-heavy track with wider sidebar:

```yaml
lab_config:
  default_layout: AssignmentRight
  sidebar_size: 42
```

Bad -- reading-heavy content crammed into a narrow sidebar:

```yaml
# Long paragraphs and explanations in a 25% panel
lab_config:
  default_layout: AssignmentRight
  sidebar_size: 25
```

### Feedback Settings

Control whether learners can provide feedback on the track:

```yaml
lab_config:
  enableFeedback: true
  feedbackRecap: true
```

### loadingMessages

Controls messages shown while the sandbox provisions. Can be `true` (default messages), `false` (no messages), or a custom list.

**Standard: set `loadingMessages: false`.** Do not author custom loading-message lists. When the sandbox is slow to provision and the learner needs engagement, use notes slides (in each challenge's `assignment.md` frontmatter) as the single loading-experience mechanism — see `loading-experience.md`.

Good -- disabled; notes slides handle the loading experience:

```yaml
lab_config:
  loadingMessages: false
```

### Theme

The `theme.name` field selects the UI theme:

```yaml
lab_config:
  theme:
    name: modern-dark
```

Common themes: `modern-dark` (recommended default), `original`.

## What to Watch For

- default_layout left as AssignmentRight when the track is a wide-dashboard walkthrough that would benefit from AssignmentBottom
- sidebar_size at default (33) when the content is clearly code-heavy (should be 30) or reading-heavy (should be 40+)
- Missing feedback settings on tracks deployed to production catalogs
- `loadingMessages` left at `true` (default) or set to a custom list -- the standard is `false`; use notes slides for loading engagement so messages and slides don't compete for attention
- Theme not specified, resulting in inconsistent appearance across an organization's track catalog

# Tab Button Usage Quality

Evaluates whether tab buttons correctly reference tab indices, use semantic variants appropriately, and create a clear visual hierarchy for learners.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Tab buttons reference wrong indices, are missing entirely, or use hardcoded tab names that break when tabs are reordered |
| 2 | Below Standard | Tab indices are mostly correct but some are off-by-one; no use of variants or block styling |
| 3 | Adequate | Tab indices are correct; buttons are present where needed, but variant and block usage is inconsistent |
| 4 | Good | All tab buttons reference correct zero-indexed tab indices; semantic variants used appropriately for context (production baseline) |
| 5 | Excellent | Button styling creates a clear visual hierarchy guiding the learner through steps -- primary actions use block=true, success for completion, warning for caution |

## Guidance

Tab buttons switch the learner's active tab in the Instruqt sandbox. They use zero-indexed references matching the order tabs appear in config.yml.

The syntax is:

```
[Button Text](tab-N)
```

where N is the zero-based index of the target tab. The first tab defined in config.yml is `tab-0`, the second is `tab-1`, and so on.

Variants add semantic color:

- **No variant** -- neutral/default, for general navigation
- **variant="success"** -- green, for completion actions or positive confirmations
- **variant="warning"** -- yellow/orange, for actions requiring caution
- **variant="danger"** -- red, for destructive or irreversible actions
- **variant="outline"** -- bordered without fill, for secondary actions

The `block=true` attribute makes the button full-width, suitable for primary call-to-action buttons.

Good -- correct zero-indexed reference with block for primary action:

```markdown
[Open the Terminal](tab-0 block=true)
```

Good -- semantic variant for a cautious action:

```markdown
[Apply the Destructive Migration](tab-1 variant="warning")
```

Good -- completion button with success variant:

```markdown
[Check Your Work](tab-0 variant="success" block=true)
```

Bad -- one-indexed tab reference (tab-1 when meaning the first tab):

```markdown
[Open the Terminal](tab-1)
```

Bad -- hardcoded tab name instead of index:

```markdown
[Open Terminal](tab-Terminal)
```

Bad -- using danger variant for a routine action:

```markdown
[View the Dashboard](tab-2 variant="danger")
```

## What to Watch For

- Tab indices are zero-based -- the first tab in config.yml is tab-0, not tab-1
- When tabs are reordered in config.yml, all tab button references in assignments must be updated
- Reserve variant="danger" for genuinely destructive actions; overuse dilutes its meaning
- Use block=true sparingly -- it works well for the primary action in a step but clutters the page if every button is full-width
- Verify tab count: if config.yml defines 3 tabs, valid references are tab-0 through tab-2
- Buttons appearing before the learner needs them cause confusion -- place tab buttons at the point in the instructions where the learner should switch

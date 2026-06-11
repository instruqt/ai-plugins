# Inline HTML Usage Quality

Evaluates whether inline HTML is used purposefully where markdown falls short, with consistent and accessible styling that enhances readability.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Raw HTML used throughout where markdown would suffice, or no HTML used where it would genuinely help (unreadable credential dumps, unstyled notes slides) |
| 2 | Below Standard | HTML present but inconsistent -- some credentials styled, others plain; iframe attributes missing; inline styles vary arbitrarily |
| 3 | Adequate | HTML used in appropriate places but styling is inconsistent or accessibility concerns exist (no alt text, tiny font sizes, low contrast) |
| 4 | Good | Inline HTML used only where markdown cannot achieve the effect; styling is consistent across the track (production baseline) |
| 5 | Excellent | HTML enhances readability with colored credentials, branded elements, and embedded content -- without cluttering the source or breaking accessibility |

## Guidance

Markdown covers most formatting needs. Reach for inline HTML only when markdown genuinely cannot do the job:

- **Colored text** for credential highlighting
- **`<iframe>`** for embedded dashboards, consoles, or documentation
- **`<style>` blocks** in notes slides for custom presentation styling
- **`<br>`** for forced line breaks where markdown paragraph spacing is too much
- **`<span>`** with inline styles for one-off formatting (step header colors, badges)

Good -- colored credential display:

```markdown
Use the following credentials to log in:

- **Username:** <code style="color: #22c55e; font-weight: bold;">admin</code>
- **Password:** <code style="color: #22c55e; font-weight: bold;">[[ Instruqt-Var key="ADMIN_PASSWORD" hostname="sandbox" ]]</code>
```

Good -- iframe for embedded content:

```markdown
<iframe src="https://[[ Instruqt-Var key="SANDBOX_ID" hostname="sandbox" ]].instruqt.io/dashboard" width="100%" height="400" frameborder="0"></iframe>
```

Good -- style block in a notes slide:

```markdown
<style>
.slide h1 { color: #3b82f6; }
.slide code { background: #1e293b; padding: 2px 6px; border-radius: 4px; }
</style>
```

Bad -- HTML used where markdown works fine:

```markdown
<p>Run the following command:</p>
<pre><code>kubectl get pods</code></pre>
```

Should be:

~~~markdown
Run the following command:

```bash,run
kubectl get pods
```
~~~

Bad -- inline style soup with no consistency:

```markdown
<span style="color:red;font-size:18px;font-weight:900;">WARNING</span>
...
<span style="color: crimson; font-size: 14px; font-weight: bold;">CAUTION</span>
```

Bad -- iframe without dimensions:

```markdown
<iframe src="https://example.com/dashboard"></iframe>
```

## What to Watch For

- If markdown can do it, use markdown -- `**bold**` over `<strong>`, `# Heading` over `<h1>`, fenced code blocks over `<pre><code>`
- Inline styles should be consistent across the track: pick one color for credentials, one approach for emphasis, and stick to it
- Iframes need explicit `width`, `height`, and `frameborder="0"` to render predictably across browsers
- Style blocks belong in notes slides, not in assignment bodies (assignments do not support `<style>` reliably)
- Color choices should maintain readability on both light and dark backgrounds -- avoid pure red (#ff0000) or very light colors
- Credential values displayed via HTML should still use Instruqt variable interpolation, not hardcoded values
- Avoid deeply nested HTML structures -- they are hard to maintain and easy to break with a missing closing tag

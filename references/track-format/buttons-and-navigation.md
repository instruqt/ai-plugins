# Buttons and Navigation

Instruqt challenge markdown supports inline buttons for tab switching, section navigation, and external links. It also supports admonitions, collapsible sections, and heading styles.

## Structure

### Button Syntax

```
[button label="LABEL" ATTRIBUTES](HREF)
```

### Admonition Syntax

```markdown
> [!TYPE]
> Content here
```

### Collapsible Syntax

```html
<details><summary>hint text</summary>content</details>
```

## Fields

### Button Attributes

| Attribute | Required | Type | Description |
|-----------|----------|------|-------------|
| `label` | Yes | string | Button text |
| `background` | No | color string | Background color (hex or named). Default: brand color |
| `color` | No | color string | Text color (hex or named) |
| `variant` | No | string | Preset style: `success` (green), `warning` (amber), `danger` (red), `outline` (outlined) |
| `block` | No | flag | Present = full-width rendering. No value needed |

`background` and `variant` are both optional. When neither is specified, the button uses the default brand color. Do not combine `background` and `variant` -- use one or the other.

### Button HREF Values

| HREF | Effect |
|------|--------|
| `tab-N` | Switch to tab N (zero-indexed: `tab-0` is the first tab) |
| `section-SLUG` | Scroll to a section heading within the current challenge |
| `section--CHALLENGE-SLUG` | Navigate to a different challenge (challenge-relative href) |
| `https://...` | Open an external URL |
| empty `()` | Navigation-only button with no target (e.g., decorative or used with check) |

### Admonition Types

| Type | Rendering |
|------|-----------|
| `[!NOTE]` | Informational callout |
| `[!WARNING]` | Warning callout |
| `[!IMPORTANT]` | Important callout |
| `[!TIP]` | Tip callout |

## Examples

### Tab Switching Buttons

Zero-indexed tab references:

```markdown
Click [button label="Terminal 1" background="#444CE7"](tab-0) to open the terminal.

Then switch to [button label="Editor" variant="success"](tab-1) to view the code.
```

### Full-Width Block Button

```markdown
[button label="Sign in" block](https://app.example.com/login)
[button label="Delete account" block variant="danger"](https://app.example.com/delete)
[button label="Continue" block background="#6c5ce7" color="#fff"](tab-0)
```

### Variant Buttons

```markdown
[button label="Done" variant="success"](tab-0)
[button label="Report" variant="warning"](https://support.example.com)
[button label="Delete" variant="danger"](tab-0)
[button label="Check" variant="outline"]()
```

### Custom Color Buttons

```markdown
[button label="Blue button" background="blue" color="white"](https://example.com)
[button label="Custom" background="#ffc814" color="#000000"](tab-0)
[button label="Purple" background="#6c5ce7"](tab-1)
```

### Section Navigation

```markdown
If you want to know more, click [button label="Details" variant="success"](section-configuration-details) to jump to that section.

Return to the top with [button label="Back to top" background="#6c5ce7"](section-introduction).
```

### Cross-Challenge Navigation

```markdown
Go to the next challenge: [button label="Next" variant="success"](section--deploy-the-application)
```

### Inline Tab Links (Non-Button)

Regular markdown links also support tab switching:

```markdown
Open the [terminal](tab-0) to continue.
```

### Admonitions

```markdown
> [!NOTE]
> When using Terraform with Google Cloud, it's recommended to use service accounts for authentication.

> [!IMPORTANT]
> Always run `terraform plan` before `terraform apply` to review changes.

> [!WARNING]
> Enabling public IP addresses for VM instances exposes them to the internet.

> [!TIP]
> Use variables to avoid hardcoding resource names.
```

### Collapsible Hint

```html
<details><summary>Click for a hint</summary>

Run `kubectl get pods` to see the current state of your deployments.

</details>
```

### Heading Styles

Both ATX and Setext styles are supported:

```markdown
## ATX Style Heading

Setext Style Heading
====================
```

### Inline HTML for Credential Display

HTML styling can be used inline for formatted credential display:

```markdown
Your password is: <span style="font-weight:bold; color:#444CE7">SuperSecret123</span>
```

## Notes

- Tab indices are zero-based: `tab-0` is the first tab, `tab-1` is the second, etc.
- `block` is a flag attribute with no value -- just include the word `block` in the attribute list.
- Empty href `()` creates a button with no navigation action, useful for decorative or instructional purposes.
- Section slugs in `section-SLUG` correspond to heading text, lowercased with spaces replaced by hyphens.
- Cross-challenge navigation uses double-dash prefix: `section--challenge-slug`.
- Admonitions use GitHub-flavored markdown alert syntax with `> [!TYPE]` on the first line.

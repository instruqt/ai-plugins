# UI Checks

UI checks verify user progress by inspecting elements within a virtual browser tab, complementing lifecycle check scripts. They confirm that a user has navigated to the correct URL or that a specific DOM element matches an expected state.

## Structure

UI checks are configured through the Instruqt web editor, not directly in YAML. They are attached to a challenge and appear under a "Checks" tab next to the Instructions tab. Once saved via the UI, they generate a check script that can be edited in code.

There are two types of UI check, and they can be combined. All conditions must pass (AND relationship).

## Fields

### URL Check

| Field | Description |
|-------|-------------|
| URL | The exact URL the user must have navigated to in the virtual browser |

### Element Check

| Field | Description |
|-------|-------------|
| Element selector | CSS path to the DOM element, selected interactively from the virtual browser |
| Expected value | The desired state or content of the selected element |
| Hint (optional) | Message shown to the user if this check fails |

## Check Behavior

- Multiple check items can be added to a single challenge (e.g., one URL check + one element check).
- All conditions must be met to pass -- they have an AND relationship.
- Once created via the UI editor, the check is stored as a script that can be edited in code from the scripts list on the track configuration page.

## Examples

### URL Check

Verifies the user navigated to a specific page:

```
URL: https://app.example.com/dashboard
```

### Element Check

Verifies a DOM element contains expected content. The element is selected interactively in the virtual browser preview, producing a CSS selector path. An optional hint guides the user on failure.

```
Element: h1.headline
Expected: "Welcome to Instruqt"
Hint: "Make sure you've navigated to the homepage and the headline is visible."
```

### Combined Check

A URL check and an element check on the same challenge. Both must pass:

```
URL check: https://app.example.com/settings
Element check: input#notifications-toggle (checked = true)
Hint: "Enable notifications in the settings page."
```

## Notes

- UI checks only work with virtual browser tabs (website services). They cannot inspect terminal or editor tabs.
- The interactive element selector in the Instruqt editor generates the CSS path automatically -- you select the element visually.
- After creation, the generated check script appears in the scripts list and can be maintained as code.

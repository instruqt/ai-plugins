# assignment.md

Defines a single challenge within a track. Each challenge lives in its own numbered directory (e.g. `01-first-challenge/assignment.md`). Contains YAML frontmatter for metadata and tabs, followed by Markdown body for instructions.

## Structure

```markdown
---
slug: my-challenge
id: <uuid-assigned-by-instruqt>
type: challenge
title: "Challenge Title"
teaser: "Short description for navigation"
difficulty: basic
timelimit: 0
enhanced_loading: null
notes:
  - type: text
    contents: |-
      # Welcome
      This is a pre-challenge slide.
  - type: video
    url: https://www.youtube.com/embed/xyz
  - type: image
    url: https://example.com/diagram.png
tabs:
  - title: Terminal
    type: terminal
    hostname: shell
  - title: Editor
    type: code
    hostname: shell
    path: /root/project
  - title: App
    type: service
    hostname: shell
    port: 8080
    path: /
lab_config:
  default_layout: AssignmentRight
  default_layout_sidebar_size: 35
---

Assignment body in Markdown goes here.
```

## Fields

### Frontmatter

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `slug` | string | required | URL-safe challenge identifier |
| `id` | UUID | — | Assigned by Instruqt, never edit manually |
| `type` | string | `challenge` | `challenge` or `quiz` |
| `title` | string | required | Display title |
| `teaser` | string | required | Short description for sidebar navigation |
| `difficulty` | string | `""` | `basic`, `intermediate`, `advanced`, or empty string |
| `timelimit` | int (seconds) | 0 | Per-challenge time limit. Usually `0` (inherits track limit) |
| `enhanced_loading` | null/bool | — | First challenge: `null` (inherit track setting). Subsequent challenges: `false` explicitly |

### Quiz type

When `type: quiz`, the challenge becomes a multiple-choice question instead of a hands-on exercise.

| Field | Type | Description |
|-------|------|-------------|
| `answers` | list of strings | Answer options displayed to the learner |
| `solution` | list of int | Correct answer indices (0-based). Single item for single-select, multiple items for multi-select. |

Single-answer quiz:

```yaml
type: quiz
title: "Knowledge Check"
answers:
  - "Option A"
  - "Option B"
  - "Option C"
solution:
  - 1
```

Multi-select quiz (learner must select all correct answers):

```yaml
type: quiz
title: "Select All That Apply"
answers:
  - "Deployments manage ReplicaSets"
  - "ReplicaSets manage Deployments"
  - "Pods are the smallest schedulable unit"
  - "Services manage Pod networking"
solution:
  - 0
  - 2
  - 3
```

### Notes (pre-challenge slides)

Array of slides shown before the challenge starts.

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | `text`, `video`, or `image` |
| `contents` | string | Markdown content (for `text` type) |
| `url` | string | URL (for `video` and `image` types) |

Notes slides require `enhanced_loading: false` at the track level to display as overlays.

Text-type notes support inline `<style>` blocks for branded styling:

```yaml
notes:
  - type: text
    contents: |-
      <style>
        .note-header { color: #326CE5; font-size: 1.5em; }
        .note-body { padding: 10px; }
      </style>
      <h1 class="note-header">Welcome</h1>
      <div class="note-body">
        This challenge covers deploying applications to Kubernetes.
      </div>
```

### Tabs

Array defining the tabs available to the learner.

#### terminal

```yaml
- title: Terminal
  type: terminal
  hostname: shell
  cmd: su - student
  workdir: /home/student/project
```

| Field | Type | Description |
|-------|------|-------------|
| `hostname` | string | Container or VM name from config.yml |
| `cmd` | string | Command to run on tab open |
| `workdir` | string | Starting directory |

#### code

```yaml
- title: Editor
  type: code
  hostname: shell
  path: /root/project
```

| Field | Type | Description |
|-------|------|-------------|
| `hostname` | string | Container or VM name |
| `path` | string | Path to file or directory to open in editor |

#### service

```yaml
- title: Web App
  type: service
  hostname: shell
  port: 8080
  path: /dashboard
  new_window: false
```

| Field | Type | Description |
|-------|------|-------------|
| `hostname` | string | Container or VM name |
| `port` | int | Port to connect to |
| `path` | string | URL path appended to base URL |
| `new_window` | bool | Open in pop-out window |
| Custom headers | — | Additional HTTP headers can be set |

#### external

```yaml
- title: Documentation
  type: external
  url: https://developer.hashicorp.com/terraform/docs
  new_window: true
```

| Field | Type | Description |
|-------|------|-------------|
| `url` | string | External URL to embed or link |
| `new_window` | bool | Open in pop-out window |

#### browser

```yaml
- title: Browser
  type: browser
  hostname: web-app
```

| Field | Type | Description |
|-------|------|-------------|
| `hostname` | string | Virtual browser name from config.yml `virtualbrowsers` section |

#### website

```yaml
- title: Dashboard
  type: website
  url: https://app-${_SANDBOX_ID}.instruqt.io
```

| Field | Type | Description |
|-------|------|-------------|
| `url` | string | HTTPS URL, supports `${_SANDBOX_ID}` interpolation |

### Challenge-level lab_config

Overrides track-level layout settings for this specific challenge.

| Field | Type | Description |
|-------|------|-------------|
| `default_layout` | string | `AssignmentLeft` or `AssignmentRight` |
| `default_layout_sidebar_size` | int | Sidebar width percentage (20-80) |
| `custom_layout` | string | JSON-encoded layout tree for precise multi-pane splits |

```yaml
lab_config:
  custom_layout: '{"type":"HSplitLayout","children":[{"type":"TabLayout","size":50},{"type":"TabLayout","size":50}]}'
```

## Body

The Markdown body after the frontmatter `---` contains the challenge instructions.

### Body features

- **Code block modifiers** — Language-tagged fenced code blocks with copy buttons
- **Tab buttons** — Inline buttons that switch to a specific tab
- **Variable interpolation** — `${VAR_NAME}` replaced at runtime
- **Admonitions** — Callout boxes for warnings, tips, notes
- **Collapsibles** — Expandable/collapsible sections for hints or optional content
- **Inline HTML** — Raw HTML for custom formatting
- **Setext headings** — Underline-style headings (`===` for H1, `---` for H2) are supported in addition to ATX (`#`) style

### Headings

Both ATX and Setext heading styles work:

```markdown
# ATX Heading (preferred)

Setext Heading
==============
```

ATX style (`#`, `##`, etc.) is preferred for consistency. Setext headings are supported but less common in Instruqt tracks.

### CDN asset URLs

Track assets (images, diagrams, downloadable files) uploaded through the Instruqt platform are served from a CDN. The URL pattern is:

```
https://play.instruqt.com/assets/tracks/<track-id>/<content-hash>/assets/<filename>
```

Reference these URLs in assignment body or notes slides. The `<content-hash>` changes when the track is updated, so URLs from previous versions may become stale.

> [!WARNING]
> **Backslash paths in asset URLs.** If track content is authored on Windows, asset paths may contain backslashes (`\`) instead of forward slashes (`/`). These cause broken images/links on the platform. Always use forward slashes in URLs, even when authoring on Windows.

### Body structure convention

Challenges typically follow this pattern:

```markdown
---
(frontmatter)
---

<style>
.step-header { color: #326CE5; font-weight: bold; }
</style>

<h2 class="step-header">Step 1: Do the first thing</h2>

Instructions for step 1.

```bash
command-to-run
```

---

<h2 class="step-header">Step 2: Do the second thing</h2>

Instructions for step 2.

---

To complete this challenge, press **Check**.
```

Key conventions:
- Colored step headers for visual structure
- Horizontal rules (`---`) between steps
- Completion marker at the end directing learner to press Check

## Examples

### Standard challenge with terminal and editor

```markdown
---
slug: deploy-application
id: abc12345-def6-7890-abcd-ef1234567890
type: challenge
title: "Deploy the Application"
teaser: "Build and deploy the containerized application"
difficulty: intermediate
timelimit: 0
enhanced_loading: null
notes:
  - type: text
    contents: |-
      # Deploying Applications
      In this challenge you will build and deploy a container.
tabs:
  - title: Terminal
    type: terminal
    hostname: shell
  - title: Editor
    type: code
    hostname: shell
    path: /root/app
  - title: App
    type: service
    hostname: shell
    port: 3000
    path: /
---

Build the Docker image and deploy it to the local cluster.

## Step 1: Build the image

Run the following command in the **Terminal** tab:

```bash
docker build -t myapp:latest .
```

---

## Step 2: Deploy to the cluster

Apply the Kubernetes manifest:

```bash
kubectl apply -f deploy.yaml
```

Check the **App** tab to verify the application is running.

---

To complete this challenge, press **Check**.
```

### Quiz challenge

```markdown
---
slug: knowledge-check
id: abc12345-def6-7890-abcd-ef1234567890
type: quiz
title: "Knowledge Check"
teaser: "Test your understanding"
answers:
  - "A Deployment manages ReplicaSets"
  - "A ReplicaSet manages Deployments"
  - "A Pod manages Deployments"
solution:
  - 0
---

Which statement about Kubernetes resource relationships is correct?
```

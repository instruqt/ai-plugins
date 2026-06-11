# Code/Editor Tab Design

Evaluates whether code editor tabs are configured to open the right files or directories, providing learners with an effective editing experience.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Editor tab fails to load, points to a nonexistent path, or opens the root filesystem with no context |
| 2 | Below Standard | Editor opens but at a directory level too high or too low -- learner must navigate extensively to find the relevant files |
| 3 | Adequate | Editor opens at a reasonable directory, but no specific file is highlighted; learner must search for the right file |
| 4 | Good | Files are accessible via correct paths; editor opens to the relevant directory or file for the challenge (production baseline) |
| 5 | Excellent | Editor experience guides the learner to exactly the right files -- single-file path for focused editing, companion pair for edit-vs-reference, or directory scoped to the relevant subtree |

## Guidance

Editor tabs open an in-browser code editor connected to the sandbox filesystem. The `path` attribute controls what the editor displays.

**Single file** -- opens the file directly for editing:

```yaml
- title: Editor
  type: code
  hostname: workstation
  path: /root/app/main.go
```

**Directory** -- opens a tree explorer rooted at that directory:

```yaml
- title: Editor
  type: code
  hostname: workstation
  path: /root/app
```

Use `workdir` to set the working directory context for the editor's integrated terminal (if available):

```yaml
- title: Editor
  type: code
  hostname: workstation
  path: /root/app
  workdir: /root/app
```

**Companion pair pattern** -- one editor tab for the file the learner edits, another for a reference file:

```yaml
- title: Edit - main.go
  type: code
  hostname: workstation
  path: /root/app/main.go

- title: Reference - schema.json
  type: code
  hostname: workstation
  path: /root/app/schema.json
```

For a full IDE experience, consider running **code-server** in the sandbox and exposing it as a service tab instead:

```yaml
- title: VS Code
  type: service
  hostname: workstation
  port: 8080
  path: /?folder=/root/app
```

Good -- single file path when the challenge focuses on one file:

```yaml
- title: Editor
  type: code
  hostname: workstation
  path: /home/student/project/config.yaml
```

Good -- directory path scoped to the relevant subtree:

```yaml
- title: Editor
  type: code
  hostname: workstation
  path: /root/terraform
```

Bad -- path to root filesystem (overwhelming, no focus):

```yaml
- title: Editor
  type: code
  hostname: workstation
  path: /
```

Bad -- nonexistent path (editor shows an error):

```yaml
- title: Editor
  type: code
  hostname: workstation
  path: /root/project/src
  # but setup script creates files in /home/student/project/src
```

Bad -- directory path when the challenge only touches one file (learner must hunt):

```yaml
- title: Editor
  type: code
  hostname: workstation
  path: /root/app
  # but the assignment only edits /root/app/deeply/nested/config.yaml
```

## What to Watch For

- `path` pointing to a single file gives the tightest focus -- use it when the challenge edits one file
- `path` pointing to a directory is better when the learner needs to see multiple files or the project structure
- The path must exist when the tab loads; if setup creates it, ensure the setup script runs before the tab is accessed
- Scope the directory as tightly as possible -- `/root/app/src` is better than `/root` when only src files matter
- The companion pair pattern (editable + reference) reduces context switching when the learner needs to consult documentation or schemas while editing
- code-server via a service tab provides a richer IDE experience (extensions, integrated terminal, Git) but requires more setup and resource overhead
- Editor tabs on Windows VMs may have path separator differences -- use forward slashes in the config regardless

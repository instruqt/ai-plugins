# Editor Tab

In-browser Monaco code editor that opens files or directories on a sandbox host. Provides syntax highlighting, basic autocomplete, and a file tree -- no extensions or integrated terminal.

## When to Use

- Learners need to edit files as part of the exercise (configuration files, source code, YAML manifests).
- The editing task is focused: one file or a small project directory.
- You want a lightweight, zero-setup editor that works immediately without installing anything on the VM.

## Config.yml Example

Directory mode (file tree on the left, editor on the right):

```yaml
tabs:
  - title: Editor
    type: code
    hostname: workstation
    path: /home/developer/app
```

Single-file mode:

```yaml
tabs:
  - title: Config
    type: code
    hostname: workstation
    path: /etc/nginx/nginx.conf
```

Companion pair (editable workspace + read-only reference):

```yaml
tabs:
  - title: Editor
    type: code
    hostname: workstation
    path: /home/developer/workspace
  - title: Reference
    type: code
    hostname: workstation
    path: /home/developer/reference
```

## Pairs With

All base compute patterns -- VMs and containers. The editor reads and writes files on the sandbox filesystem via the host specified in `hostname:`. No additional software installation is required.

## Common Patterns

- **Editor + Terminal.** The most common pairing. Learner edits code in the editor tab, switches to the terminal to run, compile, or apply it. This is the default layout for any "edit and run" exercise.
- **Companion pair.** Two editor tabs: one for the learner's working files, one showing a reference implementation or example configuration. The reference tab gives learners something to compare against without copy-pasting from the assignment text.
- **Single-file for configuration.** When the exercise involves editing one specific file (a Dockerfile, a Kubernetes manifest, an nginx config), point `path:` directly at the file so the learner does not need to navigate a directory tree.
- **Directory for project work.** When learners work across multiple files in a project, point `path:` at the project root so they can browse and switch between files.

## Common Pitfalls

- **Path does not exist at tab load time.** If the file or directory referenced by `path:` has not been created by the setup script yet, the editor tab shows an error. Ensure setup scripts create the target path before they finish.
- **Too broad a directory.** Pointing `path:` at `/` or `/home` exposes the entire filesystem. Scope it to the specific directory learners should work in.
- **Expecting IDE features.** The built-in editor is Monaco (the engine behind VS Code) but without extensions, integrated terminal, or Git integration. If learners need those, use code-server as an installed UX component instead.

## When NOT to Use

- **Learners need a full IDE experience.** If the exercise requires extensions (linters, formatters, language servers), an integrated terminal, or multi-file Git workflows, install code-server on the VM and expose it via a service tab or virtual browser.
- **No file editing in the exercise.** If the track is purely CLI-based and learners never edit files, omit the editor tab to keep the interface focused.
- **Binary or non-text files.** The editor handles text files. For image editing, database GUIs, or other non-text workflows, use the appropriate tool via a service tab.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Kubernetes manifest authoring (editor for YAML + terminal for kubectl apply)
- Web development basics (editor for HTML/CSS/JS + service tab showing the live page)
- Configuration management (single-file editor for a config file + terminal to validate and apply)
- Code review exercise (companion pair: learner's code + reference implementation)

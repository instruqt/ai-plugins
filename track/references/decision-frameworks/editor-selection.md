# Editor Selection

Choosing between the native Instruqt editor tab and code-server when a track requires learners to edit files.

## Comparison

| Aspect | Native Editor Tab | Code-Server |
|--------|------------------|-------------|
| Type | `type: code` in assignment.md | `type: service` pointing at port 8443 |
| Setup | None (built-in Instruqt feature) | Install code-server, configure systemd, create service tab |
| Infrastructure | Any host (container or VM) | VM only (needs systemd for reliable operation) |
| Editor | Monaco (same engine as VS Code) | Full VS Code (code-server fork) |
| Extensions | None | Full VS Code marketplace |
| Integrated terminal | No | Yes |
| Git integration | No | Yes |
| Multi-file navigation | Directory tree mode only | Full file explorer, search, go-to-definition |
| File targeting | Single file or directory via `path:` | Workspace directory via URL parameter |
| Resource cost | Zero (runs in the Instruqt player) | ~200-500 MB on the VM |

## Decision Criteria

### Use the native editor tab when:

- Learner edits one or a few specific files (config files, single scripts).
- No extensions, IntelliSense, or integrated terminal are needed.
- Track runs on containers (code-server is not available).
- Minimal setup is preferred — the editor tab just works.
- The companion pair pattern (edit file + reference file) is sufficient.

### Use code-server when:

- Learner works across multiple files in a project (application development, Terraform modules).
- Extensions are valuable (Python, Terraform, YAML language support, linting).
- Integrated terminal inside the editor improves the workflow.
- Git integration is part of the exercise.
- The track teaches IDE-based development workflows.

## Trade-offs

| Factor | Native Editor | Code-Server |
|--------|--------------|-------------|
| Setup time | Zero | 30-60 seconds (install + start) |
| Learner familiarity | Basic editor, minimal learning curve | VS Code, very familiar to developers |
| Focus | Tight — opens exactly the file(s) you specify | Broad — full workspace access |
| Infrastructure overhead | None | VM required, ~200-500 MB RAM |
| Reliability | Built-in, always works | Service must start and stay running |
| Customization | Path only | Themes, extensions, settings, keybindings |

## Common Mistakes

- **Using code-server for a single config file edit.** The native editor tab is sufficient and requires no setup. Code-server adds complexity for no benefit when the learner only edits one file.
- **Using the native editor for a full development workshop.** If learners write code across multiple files, need IntelliSense, or use the integrated terminal, the native editor's limitations will frustrate them.
- **Forgetting that native editor works on containers.** Code-server requires a VM. If the track uses containers, the native editor is the only option.
- **Not pre-configuring code-server settings.** Without `security.workspace.trust.enabled: false` and a dark theme, learners get a trust dialog and a light theme that feels inconsistent with the Instruqt dark player.

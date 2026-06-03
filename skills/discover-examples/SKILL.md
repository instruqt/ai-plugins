# Discover Examples

Find relevant infrastructure pattern and decision framework files for a given query.

## Progress reporting

Do not create your own top-level task list. The invoking command owns the user-facing task list. If this skill represents a single user-visible step in that list, your work maps to one entry. If you need finer-grained progress, update the existing task list rather than starting a new one.

## Paths

- Infrastructure: `${CLAUDE_PLUGIN_ROOT}/references/infrastructure/_index.md`
- Decision frameworks: `${CLAUDE_PLUGIN_ROOT}/references/decision-frameworks/_index.md`

## Workflow

1. Read both manifest files
2. Match the query against file descriptions — use keyword overlap, synonyms, and related concepts
3. Return matching file paths with their one-line descriptions
4. The invoking agent reads the files it needs

## Scope

This skill covers:
- Infrastructure (`references/infrastructure/`) — base compute, cloud accounts, managed Kubernetes, runtime provisioning, learner UX, modifiers, sandbox presets
- Decision frameworks (`references/decision-frameworks/`) — trade-off analysis for common choices

## Usage

Agent invokes: "I need to set up a track with a web IDE"
Skill returns:
```
- infrastructure/learner-ux/installed/code-server.md — VS Code in browser via code-server
- infrastructure/learner-ux/installed/kasmvnc.md — Desktop apps via KasmVNC (alternative for non-web IDEs)
- infrastructure/learner-ux/native/editor-tab.md — Native editor tab for lightweight file editing
- decision-frameworks/editor-selection.md — Native editor tab vs code-server
- decision-frameworks/custom-image-vs-heredoc.md — Build time vs iteration speed trade-off
```

Agent invokes: "Should I use a sandbox preset or custom config?"
Skill returns:
```
- infrastructure/sandbox-presets.md — Pre-built environments, no config.yml needed
- decision-frameworks/preset-vs-custom-config.md — Capabilities comparison and recommendation
```

Agent invokes: "I need AWS with Kubernetes"
Skill returns:
```
- infrastructure/cloud-accounts/container-plus-aws.md — cloud-client + AWS account
- infrastructure/managed-kubernetes/container-plus-eks.md — cloud-client + AWS + EKS
- infrastructure/base-compute/single-vm-kind-k3s.md — Local Kubernetes via Kind/k3s on VM
- decision-frameworks/container-vs-vm.md — Container vs VM selection
- decision-frameworks/cloud-provider-selection.md — Cloud provider configuration guidance
```

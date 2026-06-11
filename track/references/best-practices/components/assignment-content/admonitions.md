# Callout and Admonition Quality

Evaluates whether callout blocks are used appropriately to highlight important information without overuse, and whether the correct type is chosen for each context.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No callouts used where they are clearly needed (destructive commands without warning), or callouts used for every paragraph |
| 2 | Below Standard | Callouts present but wrong types chosen (NOTE for destructive warnings, WARNING for helpful tips) |
| 3 | Adequate | Callouts generally match context but some are mistyped or there are too many in a single challenge |
| 4 | Good | Callouts used where genuinely needed with correct types; not overused (production baseline) |
| 5 | Excellent | Callouts enhance scanning and draw attention to exactly the right moments -- warnings before danger, tips for efficiency, notes for context |

## Guidance

Instruqt assignments support GitHub-flavored admonition syntax:

```markdown
> [!NOTE]
> Supplementary context the learner might find useful but can skip without consequence.

> [!TIP]
> Optional improvement or shortcut that makes the task easier.

> [!IMPORTANT]
> Key information the learner must understand for the task to succeed.

> [!WARNING]
> Action that could cause problems if done incorrectly -- data loss, broken state, cost implications.
```

Choose the type based on consequence, not emphasis:

| Type | When to use |
|------|-------------|
| NOTE | Background context, version notes, "for your information" |
| TIP | Optional shortcuts, editor tricks, efficiency suggestions |
| IMPORTANT | Required knowledge that is easy to miss in the flow of instructions |
| WARNING | Destructive actions, irreversible changes, cost-incurring operations |

Good -- warning before a destructive command:

```markdown
> [!WARNING]
> The following command deletes all resources in the namespace. This cannot be undone.

```bash,copy
kubectl delete namespace lab-environment
```
```

Good -- tip for an optional shortcut:

```markdown
> [!TIP]
> You can use `kubectl get all -n default` to verify all resources in a single command.
```

Good -- note providing background context:

```markdown
> [!NOTE]
> This lab uses Kubernetes 1.28. Some commands may differ in earlier versions.
```

Bad -- warning used for non-dangerous information:

```markdown
> [!WARNING]
> The dashboard may take a few seconds to load.
```

Bad -- multiple callouts stacked back-to-back:

```markdown
> [!NOTE]
> Helm is a package manager for Kubernetes.

> [!TIP]
> You can also install charts from OCI registries.

> [!IMPORTANT]
> Make sure Helm is installed before continuing.
```

Bad -- no callout before a destructive operation:

```markdown
Run the following command to reset the cluster:

```bash,run
kubeadm reset --force
```
```

## What to Watch For

- More than two callouts in a single challenge step overwhelms the learner -- consolidate or remove the least critical
- WARNING should appear before the dangerous command, not after
- Do not use callouts as a substitute for clear writing -- if the main text is well-written, most information does not need a callout
- NOTE is the most overused type; ask whether the information truly needs to stand apart from the surrounding text
- IMPORTANT should be reserved for things the learner will miss if they skim -- not for restating what the instructions already say
- Avoid nesting code blocks inside callouts unless the callout specifically warns about that code

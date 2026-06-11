# Collapsible Hint Quality

Evaluates whether collapsible sections are used effectively to provide opt-in help without spoiling answers or cluttering the main instruction flow.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No hints provided for difficult steps, or answers are visible in the main body with no way to attempt independently |
| 2 | Below Standard | Hints exist but are not collapsible -- answers are immediately visible, defeating the purpose |
| 3 | Adequate | Collapsible `<details>` blocks present but inconsistently styled, or used for non-hint content that should be in the main body |
| 4 | Good | Hints available for tricky steps using properly styled collapsibles; answers hidden by default (production baseline) |
| 5 | Excellent | Hint system creates scaffolded learning -- struggling learners get progressive help without penalizing confident ones; hints are layered from nudge to full answer |

## Guidance

Use HTML `<details>` and `<summary>` elements to create collapsible hint blocks. These are collapsed by default, requiring the learner to actively choose to reveal them.

Good -- basic hint for a challenging step:

```markdown
<details>
<summary>Hint: Not sure which resource type to use?</summary>

The Kubernetes resource for running a one-time task is a **Job**. Use `kind: Job` in your manifest.

</details>
```

Good -- layered hints (nudge then answer):

```markdown
<details>
<summary>Hint 1: Where to look</summary>

Check the pod logs for the error message. The namespace is `monitoring`.

</details>

<details>
<summary>Hint 2: Full solution</summary>

```bash,copy
kubectl logs -n monitoring deploy/alertmanager | grep "config error"
```

The configuration file has a typo on line 12. Fix `recievers` to `receivers`.

</details>
```

Good -- styled summary for visibility:

```markdown
<details>
<summary><strong>Need help?</strong> Click to reveal a hint.</summary>

The service account needs the `cluster-admin` ClusterRole bound via a ClusterRoleBinding, not a RoleBinding.

</details>
```

Bad -- hint contains only "try harder" with no useful guidance:

```markdown
<details>
<summary>Hint</summary>

Think about what you learned earlier.

</details>
```

Bad -- critical instruction hidden inside a collapsible:

```markdown
<details>
<summary>Important configuration</summary>

Set the `replicas` field to 3. Without this, the check script will fail.

</details>
```

Bad -- collapsible used for routine content that should be visible:

```markdown
<details>
<summary>Step 2 commands</summary>

```bash,run
kubectl apply -f manifest.yaml
```

</details>
```

## What to Watch For

- Never hide required instructions inside collapsibles -- only supplementary help and answers belong there
- A blank line is required after `<summary>` and before `</details>` for markdown inside the block to render correctly
- Hints should be actionable -- tell the learner what to look at or what concept to apply, not just "try again"
- Layered hints (nudge, then detailed help, then full answer) work well for exercises where independent problem-solving is the goal
- Overusing collapsibles makes the page feel sparse -- if every step has a hint, the challenge may be too difficult or under-explained
- Keep summary text descriptive ("Hint: Not sure which port to use?") rather than generic ("Click here")

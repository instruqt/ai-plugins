# Step-Based Body Structure Quality

Evaluates whether the assignment body uses consistent numbered steps, clear section separation, and completion markers to guide the learner through the challenge.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Wall of text with no step numbering, no visual separation, and no indication of when the challenge is complete |
| 2 | Below Standard | Some numbered steps but inconsistent formatting; no completion marker; sections run together |
| 3 | Adequate | Numbered steps present with some separation, but completion section is missing or formatting is inconsistent across challenges |
| 4 | Good | Clear step progression with horizontal rules between sections; completion markers present; consistent formatting (production baseline) |
| 5 | Excellent | Step structure creates a natural rhythm -- pre-work is separated, steps are scannable, completion marker is unmistakable, and the learner always knows where they are |

## Guidance

A well-structured assignment body follows this pattern:

1. **Context section** -- brief orientation (what the learner will do and why)
2. **Numbered steps** -- each step is a discrete action, separated by horizontal rules
3. **Completion marker** -- a clear signal that the challenge is done, often with a pointer to the next challenge

Good -- complete step structure:

```markdown
In this challenge you will deploy a sample application to the Kubernetes cluster and verify it is running.

---

## Step 1: Apply the Deployment

Run the following command to create the deployment:

```bash,run
kubectl apply -f /root/manifests/app.yaml
```

---

## Step 2: Verify the Pods

Check that the pods are running:

```bash,run
kubectl get pods -n app
```

You should see two pods with status `Running`.

---

## Completion

Once both pods show `Running` status, you have completed this challenge.

Click **Check** to continue to the next challenge.
```

Good -- colored step headers (optional convention):

```markdown
## <span style="color: #f97316;">Step 1: Configure the Database</span>
```

Good -- completion marker with checkmark:

```markdown
---

To complete this challenge, press **Check**.
```

Bad -- no step separation:

```markdown
First run kubectl apply. Then check the pods. After that open the dashboard and click Settings. Now configure the alert rule by entering the threshold value.
```

Bad -- missing completion signal:

```markdown
## Step 3: View the Logs

```bash,run
kubectl logs -f deploy/app
```

(assignment ends abruptly)
```

Bad -- inconsistent heading levels:

```markdown
# Step 1: Do This
### Step 2: Do That
## Step 3: And This
```

## What to Watch For

- Every challenge should end with an explicit completion marker telling the learner what triggers the check
- Horizontal rules (`---`) between major steps create visual breathing room -- without them, long challenges become a wall of text
- Step numbering should be consistent: either all `## Step N:` headings or all numbered list items, not a mix
- Context or pre-work sections (background info, credential display) should be visually separated from the action steps
- If the challenge has more than 5-6 steps, consider whether it should be split into multiple challenges
- The completion marker should tell the learner what state to be in, not just say "click Check" -- this helps them self-diagnose before running the check script

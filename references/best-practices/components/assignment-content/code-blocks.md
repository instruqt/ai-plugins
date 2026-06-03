# Code Block Usage Quality

Evaluates whether code blocks use correct modifiers, language tags, and placeholder handling to create a smooth learner experience.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Code blocks lack language tags and modifiers entirely, or use invalid syntax like ```bash {"run": "execute"} |
| 2 | Below Standard | Some blocks have language tags but modifiers are missing or wrong; runnable commands marked as copy, placeholder blocks marked as run |
| 3 | Adequate | Most blocks have language tags and modifiers, but a few are inconsistent or use the wrong modifier for the context |
| 4 | Good | All code blocks have appropriate modifiers; runnable blocks use correct language tags; placeholder blocks use copy modifier (production baseline) |
| 5 | Excellent | Modifiers chosen to optimize learner workflow -- run for safe idempotent commands, copy for blocks requiring edits, nocopy for display-only reference code |

## Guidance

Every fenced code block in an assignment should declare both a language tag and the correct Instruqt modifier. The three modifiers are:

- **run** -- the learner clicks to execute the command in the terminal tab. Use only for safe, idempotent commands that need no editing.
- **copy** -- the learner clicks to copy the block to clipboard. Use for blocks that contain placeholders the learner must edit, or for multi-step commands where the learner should review before running.
- **nocopy** -- the block is display-only. Use for expected output, reference snippets, or diffs the learner should read but not execute.

Good -- safe command with run modifier:

~~~markdown
```bash,run
kubectl get pods -n default
```
~~~

Good -- placeholder block with copy modifier:

~~~markdown
```bash,copy
curl -X POST https://api.example.com/documents/<DOCUMENT_ID>/publish
```
~~~

Good -- expected output with nocopy:

~~~markdown
```text,nocopy
NAME       READY   STATUS    RESTARTS   AGE
nginx      1/1     Running   0          2m
```
~~~

Bad -- invalid JSON-style modifier syntax:

~~~markdown
```bash {"run": "execute"}
kubectl get pods
```
~~~

Bad -- placeholder block with run modifier (learner executes unexpanded placeholder):

~~~markdown
```bash,run
export API_KEY=<YOUR_API_KEY>
```
~~~

Bad -- missing language tag:

~~~markdown
```
kubectl get pods
```
~~~

## What to Watch For

- Blocks containing angle-bracket placeholders (`<DOCUMENT_ID>`, `<YOUR_NAME>`) must never use the run modifier
- Shell commands that modify state (delete, apply, create) should generally use copy so the learner can review before executing
- Output blocks should use nocopy to prevent learners from accidentally pasting terminal output
- Every code block needs a language tag -- even plain text should use `text`
- The run modifier only works with terminal-compatible language tags (bash, sh, powershell)

# Python Run for Non-Python Content

Using the ` ```python,run ` code fence for content that is not Python code.

## Why It's Bad

The `python,run` annotation tells Instruqt to execute the block as Python. If the content is actually a shell command or language-agnostic snippet, it will either error out or produce unexpected behavior. It also confuses learners about what language they are working with.

## What to Do Instead

Match the code fence annotation to the content:

- ` ```bash,run ` -- for shell commands the learner should execute.
- ` ```run ` -- for language-agnostic commands.
- ` ```python ` (no `,run`) -- for display-only Python snippets the learner reads but does not execute inline.
- ` ```python,run ` -- only for Python code that should actually be executed.

## Examples

Bad -- shell command marked as python,run:

````markdown
```python,run
kubectl get pods -n default
```
````

Good -- shell command with correct annotation:

````markdown
```bash,run
kubectl get pods -n default
```
````

Bad -- display-only Python marked as executable:

````markdown
```python,run
# This is just for reference
def example():
    return "hello"
```
````

Good -- display-only Python without run:

````markdown
```python
# This is just for reference
def example():
    return "hello"
```
````

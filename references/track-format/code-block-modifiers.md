# Code Block Modifiers

Fenced code blocks in challenge markdown support modifiers that control rendering behavior: auto-run on click, copy buttons, line wrapping, and line numbers. Modifiers are appended to the opening fence after the language tag.

## Structure

Basic syntax:

````
```language,modifier1,modifier2
code here
```
````

Modifiers are comma-separated and follow the language identifier (if present). Some modifiers work without a language tag.

## Fields

### Modifiers

| Modifier | Effect |
|----------|--------|
| `run` | Auto-runs the command in the terminal when the user clicks it |
| `copy` | Adds a copy-to-clipboard button |
| `nocopy` | Removes the copy button (suppresses default copy behavior) |
| `wrap` | Wraps long lines instead of horizontal scrolling |
| `line-numbers` | Shows line numbers in a gutter |

### Supported Languages

Common language tags: `aql`, `bash`, `sh`, `shell`, `click`, `dockerfile`, `esql`, `graphql`, `hcl`, `http`, `js`, `javascript`, `json`, `logs`, `markdown`, `md`, `mongodb`, `plaintext`, `python`, `ruby`, `text`

Notes on language synonyms:
- `shell` and `sh` are synonyms for `bash`
- `mongodb` with `run` works for mongosh auto-execution
- `javascript,run` works for mongosh auto-execution

## Modifier Combinations

Modifiers can be combined freely:

| Combination | Use case |
|-------------|----------|
| `bash,run` | Auto-run a bash command on click |
| `run` | Auto-run without specifying a language |
| `copy` | Copy button, works without a language tag |
| `nocopy` | Suppress copy button |
| `bash,wrap,copy` | Wrapping + copy button |
| `bash,line-numbers` | Line numbers in gutter |
| `nocopy,run` | Auto-run without copy button |
| `nocopy,wrap` | Wrap without copy button |
| `copy,wrap` | Copy button + line wrapping |
| `bash,nocopy` | Bash highlighting, no copy button |
| `text,nocopy` | Plain text, no copy button |
| `python,nocopy` | Python highlighting, no copy button |
| `hcl,copy` | HCL highlighting with copy button |

## Examples

### Auto-Run Command

The user clicks the code block to execute it in the terminal:

````
```bash,run
mkdir instruqt
```
````

### Language-Less Run Block

No syntax highlighting, still runnable:

````
```run
terraform apply -auto-approve
```
````

### Copy-Only Block (Preferred for Placeholders)

For commands with placeholders the user must fill in, use `copy` instead of `run`:

````
```copy
curl https://api.example.com/v1/resources/<DOCUMENT_ID>
```
````

### Wrapped Long Output

````
```bash,wrap
echo "This is a very long line of code that will now wrap to the next line instead of scrolling horizontally."
```
````

### Line Numbers

````
```bash,line-numbers
mkdir instruqt
cd instruqt
ls
```
````

### No Copy Button

````
```bash,nocopy
mkdir instruqt
```
````

## Notes

- **Invalid syntax**: ```` ```bash {"run": "execute"} ```` is NOT valid. It renders as a plain code block. Use comma-separated modifiers only.
- **Placeholder blocks**: Code containing placeholders like `<DOCUMENT_ID>` should use plain ```` ```bash ```` or ```` ```copy ````, never ```` ```bash,run ````. Users need to edit the values before running.
- **Preferred for placeholders**: ```` ```copy ```` provides a copy button without auto-run behavior, which is ideal for commands the user needs to modify.
- Modifiers are rendering hints -- they do not affect the actual code content.

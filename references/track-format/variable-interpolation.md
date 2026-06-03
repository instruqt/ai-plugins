# Variable Interpolation

Instruqt variables let scripts inject dynamic values into challenge content at render time. Variables are set with the `agent` CLI in lifecycle scripts and surfaced in markdown using a shortcode syntax.

## Structure

### Setting Variables (in scripts)

```bash
agent variable set KEY VALUE
```

### Reading Variables (in scripts)

```bash
agent variable get KEY
```

### Rendering Variables (in markdown)

```
[[ Instruqt-Var key="KEY_NAME" hostname="HOST_NAME" ]]
```

The `hostname` parameter specifies which sandbox host the variable was set on. Variables are resolved before the content is parsed.

## Fields

| Parameter | Required | Description |
|-----------|----------|-------------|
| `key` | Yes | The variable name, matching what was passed to `agent variable set` |
| `hostname` | Yes | The sandbox host name where the variable was set |

## Supported Locations

The interpolation shortcode works inside ANY string field in challenge content:

- `cmd:` field values
- `url:` field values
- `path:` field values
- Frontmatter `teaser` and `description`
- Body markdown (including inside fenced code blocks)
- `notes` text
- Image alt-text
- Button labels
- Table cells

Variables are resolved before content is parsed, so no escaping is needed.

## Examples

### Basic: Display a Dynamic Endpoint

Setup script (`setup-workstation`):

```bash
agent variable set DB_ENDPOINT "db.internal.example.com:5432"
```

Challenge markdown:

```markdown
Connect to the database at:

[[ Instruqt-Var key="DB_ENDPOINT" hostname="workstation" ]]
```

### Bridge Cloud Credentials to Markdown

Cloud provider credentials are auto-injected as environment variables in sandbox hosts. To surface them in challenge content, bridge them with `agent variable set`:

Setup script:

```bash
agent variable set PROJECT_ID "$INSTRUQT_GCP_PROJECT_<RESOURCE>_PROJECT_ID"
agent variable set CONSOLE_URL "https://console.cloud.google.com/home/dashboard?project=$INSTRUQT_GCP_PROJECT_<RESOURCE>_PROJECT_ID"
```

Challenge markdown:

```markdown
Open the [button label="GCP Console" variant="success"]([[ Instruqt-Var key="CONSOLE_URL" hostname="workstation" ]])

Your project ID is: `[[ Instruqt-Var key="PROJECT_ID" hostname="workstation" ]]`
```

### Expose _SANDBOX_ID

`_SANDBOX_ID` is NOT in the variable store by default. It must be bridged explicitly in the track-level setup script:

```bash
agent variable set SANDBOX_ID "$_SANDBOX_ID"
```

Then in markdown:

```markdown
Sandbox ID: [[ Instruqt-Var key="SANDBOX_ID" hostname="workstation" ]]
```

### Inside a Code Block

Variables resolve even inside fenced code blocks:

````markdown
```bash,copy
gcloud config set project [[ Instruqt-Var key="PROJECT_ID" hostname="workstation" ]]
```
````

### Org-Specific Header Variables

Some organizations emit fixed header variables that can be referenced directly:

- `ERROR_MSG`
- `BADGE_INFO`
- `RESTORE_PROGRESS`

## Notes

- Variables are resolved before markdown parsing -- no escaping or special handling needed.
- Auto-injected cloud credential environment variables (e.g., `INSTRUQT_AWS_ACCOUNT_*`, `INSTRUQT_GCP_PROJECT_*`, `INSTRUQT_AZURE_SUBSCRIPTION_*`) are available in scripts but must be bridged to the variable store with `agent variable set` to use them in markdown interpolation.
- `_SANDBOX_ID` is a special case: it exists as an environment variable but is NOT automatically in the variable store. Always bridge it explicitly if you need it in content.
- `agent variable set` and `agent variable get` work in any lifecycle script (setup, check, solve, cleanup).

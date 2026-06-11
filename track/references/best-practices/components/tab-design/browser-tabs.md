# Browser and Website Tab Design

Evaluates whether browser and website tabs are configured with the correct type, URL scheme, and variable interpolation for cloud consoles, external sites, and embedded forms.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Wrong tab type chosen (browser vs website); URLs fail to load or use HTTP for external sites |
| 2 | Below Standard | Tab type is correct but URL is hardcoded where interpolation is needed, or external URL is HTTP instead of HTTPS |
| 3 | Adequate | Correct tab type and URLs work, but no variable interpolation or deep linking where it would improve the experience |
| 4 | Good | Correct tab type for each use case; URLs work; variable interpolation used where needed (production baseline) |
| 5 | Excellent | Browser tabs are fully integrated into the learning flow -- cloud consoles pre-authenticated, forms capture lead data, deep links land on the exact resource |

## Guidance

Instruqt supports two tab types for displaying web content from outside the sandbox:

**`browser`** -- renders a virtual browser (backed by virtualbrowsers) inside the sandbox. Used for cloud provider consoles (AWS, GCP, Azure) that require full browser capabilities:

```yaml
- title: AWS Console
  type: browser
  hostname: aws
  path: https://console.aws.amazon.com/ec2
```

**`website`** -- embeds an external HTTPS URL directly in an iframe. Used for documentation, forms, or any public HTTPS site:

```yaml
- title: Documentation
  type: website
  url: https://docs.example.com/getting-started
```

Variable interpolation is available for dynamic URLs:

- `${_SANDBOX_ID}` -- unique sandbox identifier
- `${INSTRUQT_PARTICIPANT_ID}` -- unique participant identifier

```yaml
- title: Feedback Form
  type: website
  url: https://forms.example.com/survey?sandbox=${_SANDBOX_ID}&participant=${INSTRUQT_PARTICIPANT_ID}
```

**HubSpot/Typeform embedding** for lead capture on SE demo tracks:

```yaml
- title: Get Started
  type: website
  url: https://share.hsforms.com/1abc123?sandbox_id=${_SANDBOX_ID}
```

Good -- virtual browser for a cloud console:

```yaml
- title: GCP Console
  type: browser
  hostname: gcp
  path: https://console.cloud.google.com/kubernetes/list
```

Good -- website tab for external documentation:

```yaml
- title: Docs
  type: website
  url: https://developer.hashicorp.com/terraform/tutorials
```

Good -- lead capture form with participant tracking:

```yaml
- title: Register
  type: website
  url: https://share.hsforms.com/1xyz?participant_id=${INSTRUQT_PARTICIPANT_ID}
```

Bad -- using website type for a cloud console (needs full browser capabilities):

```yaml
- title: AWS Console
  type: website
  url: https://console.aws.amazon.com
```

Bad -- HTTP URL in a website tab (must be HTTPS):

```yaml
- title: Docs
  type: website
  url: http://docs.example.com
```

Bad -- using browser type for a simple documentation site (wastes resources):

```yaml
- title: Docs
  type: browser
  hostname: docs
  path: https://docs.example.com
```

Bad -- hardcoded sandbox ID instead of interpolation:

```yaml
- title: Dashboard
  type: website
  url: https://app.example.com/sandbox/abc123
```

## What to Watch For

- `browser` type requires a virtualbrowsers-backed hostname and uses `path` for the URL; `website` type uses `url` directly
- `website` URLs must be HTTPS -- HTTP will be blocked by the browser's mixed-content policy
- Cloud provider consoles (AWS, GCP, Azure) require `browser` type because they use complex JavaScript, cookies, and redirects that do not work in a simple iframe
- `${_SANDBOX_ID}` and `${INSTRUQT_PARTICIPANT_ID}` are interpolated at render time -- they work in `url` and `path` attributes
- HubSpot and Typeform forms embed well as `website` tabs for lead capture in SE demo tracks; pass participant ID for attribution
- Virtual browser tabs consume more resources than website tabs -- only use `browser` when iframe embedding is insufficient
- Website tabs are subject to the target site's CSP and X-Frame-Options headers -- if the site blocks iframing, it will not render (consider `new_window: true` on a service tab or a different approach)

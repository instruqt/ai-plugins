# Virtual Browser (External URL)

Full Chromium browser rendered as a tab in the Instruqt player, pointing at an external URL -- a SaaS product, cloud console, or third-party portal outside the sandbox. Use when learners need to interact with a real-world service through a browser.

## When to Use

- Learners need to sign in to and interact with a cloud console (AWS, Azure, GCP).
- The track provisions a SaaS tenant or external service and learners use its web UI.
- The external site requires cookies, JavaScript, popups, or OAuth -- making `type: website` (iframe) unreliable.

## Config.yml Example

Cloud console with a static URL:

```yaml
# config.yml
virtualbrowsers:
  - name: cloud-console
    url: https://console.cloud.google.com
```

SaaS portal with a URL constructed from an agent variable:

```yaml
# config.yml
virtualbrowsers:
  - name: saas-portal
    url: https://TENANT_SLUG.app.example.com
```

The `TENANT_SLUG` value would be set by a setup script using `agent variable set`, and the virtual browser resolves it at launch.

```yaml
# assignment.md frontmatter
tabs:
  - title: Terminal
    type: terminal
    hostname: workstation
  - title: Cloud Console
    type: browser
    hostname: cloud-console
```

## Pairs With

- **Cloud accounts (AWS/Azure/GCP).** The most common pairing. The track provisions cloud credentials, and the virtual browser gives the learner a console to use them.
- **Account pool checkout.** Tracks that check out pre-provisioned SaaS tenant credentials pair those with a virtual browser pointing at the SaaS login page.
- **Any base compute pattern.** The virtual browser is independent of sandbox compute -- it runs its own Chromium instance. The sandbox VM or container is used for CLI work alongside the external browser.

## Common Patterns

- **Cloud console + Terminal.** Learner uses the terminal for CLI commands (`aws`, `gcloud`, `az`) and the virtual browser for console-based verification or GUI operations. The most common layout for cloud infrastructure tracks.
- **SaaS product walkthrough.** Learner follows guided steps in an external product UI. Credentials are injected via setup scripts and displayed in the assignment or a note tab.
- **Multi-console tracks.** Some tracks need both a cloud console and a separate SaaS portal. Define multiple `virtualbrowsers:` entries, each with its own external URL.

## Common Pitfalls

- **Credentials not ready.** If the virtual browser loads a login page before setup scripts have finished provisioning credentials, the learner has to wait or refresh. Ensure credential provisioning completes in the setup script and display credentials clearly in the assignment.
- **URL changes between challenges.** If different challenges need the virtual browser to point at different URLs, you may need to update the virtual browser URL in lifecycle scripts or use a URL that covers the full workflow.
- **External site blocks headless browsers.** Some SaaS products detect and block headless Chromium. This is uncommon but can cause unexpected failures during testing.
- **Higher resource cost.** Like sandbox virtual browsers, each external virtual browser runs a Chromium instance. Use `type: website` or `type: external` if the site works in an iframe.

## When NOT to Use

- **The external site works in an iframe.** If the site allows iframe embedding (no restrictive CSP, no popup flows), use `type: website` or `type: external` instead -- they are lighter.
- **The target is inside the sandbox.** For web applications running on sandbox hosts, use a service tab or a sandbox virtual browser (see `virtual-browser-sandbox.md`).
- **Read-only documentation links.** For linking to external docs the learner just reads, use `type: external` with `new_window: true`.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Cloud infrastructure provisioning with AWS/Azure/GCP console and CLI
- SaaS administration walkthrough using a pre-provisioned tenant
- CI/CD platform configuration in a hosted service (GitHub, GitLab SaaS)
- Multi-cloud workshop with separate console tabs for two cloud providers

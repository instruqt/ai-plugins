# Tab Type Selection

Choosing the right tab type for presenting content and interfaces to the learner.

## Tab Types at a Glance

| Tab Type | What It Does | Best For |
|----------|-------------|----------|
| `terminal` | Shell on a sandbox host | CLI exercises, command execution |
| `code` | Monaco editor on sandbox filesystem | Editing config files, scripts, code |
| `service` | Iframe proxy to hostname:port | Web UIs running inside the sandbox |
| `website` | Iframe with direct HTTPS URL | External documentation, sandbox-public URLs |
| `external` | External URL tab | Links to docs, references |
| `browser` | Full Chromium via virtualbrowsers | Cloud consoles, SPAs that break in iframes |

## Decision Flow

```
Learner needs to run commands?
  --> terminal tab

Learner needs to edit files?
  --> code tab (or code-server via service tab -- see editor-selection.md)

Learner needs to see a web UI running in the sandbox?
  Does it work in an iframe?
    Yes --> service tab
    No (CSP blocks it, needs popups, OAuth redirects)
      Can you strip headers with custom_response_headers?
        Yes --> service tab with header overrides
        No --> virtual browser (sandbox)

Learner needs to interact with an external system (SaaS, cloud console)?
  Needs full browser (login, popups, JavaScript-heavy)?
    --> virtual browser (external)
  Just needs to view a page?
    --> website or external tab

Learner needs a full GUI desktop?
  --> See gui-access-method.md
```

## Combining Tabs

Most tracks use multiple tab types. Common combinations:

| Combination | Use Case |
|-------------|----------|
| Terminal + Service | CLI exercises with a web UI to observe results |
| Terminal + Code | Edit files, then run commands to apply them |
| Terminal + Code + Service | Full dev workflow: edit, run, observe |
| Terminal + Browser | CLI exercises with a cloud console for verification |
| Terminal + Service + Browser | Infrastructure exercises with local services + cloud console |

## Service Tab vs Website Tab vs Virtual Browser

This is the most common source of confusion.

| Question | Service | Website | Virtual Browser |
|----------|---------|---------|----------------|
| Where does the app run? | Inside sandbox | External | Either |
| How is the URL constructed? | hostname + port + path | Explicit URL | Explicit URL |
| Iframe or real browser? | Iframe (proxied) | Iframe (direct) | Real Chromium |
| CSP control? | Yes (`custom_response_headers`) | No | N/A (real browser) |
| Can handle OAuth/popups? | No | No | Yes |
| Resource overhead | None | None | ~512 MB (browser container) |

**Rule of thumb:** Start with a service tab. If the iframe is blank or broken, try adding `custom_response_headers` to strip CSP. If that doesn't work, escalate to a virtual browser. Use `website` only for external URLs that serve permissive headers.

## Common Mistakes

- **Using a virtual browser when a service tab would work.** Virtual browsers consume ~512 MB and add complexity. Most web UIs work fine in a service tab, especially with CSP header overrides. Only escalate when necessary.
- **Using `website` for in-sandbox services.** The `website` tab type expects an external HTTPS URL. For services running inside the sandbox, use `service` — it handles the proxy routing automatically.
- **Forgetting the terminal tab.** Even tracks focused on web UIs usually need a terminal tab for setup verification, debugging, or CLI commands. Always include at least one.
- **Too many tabs.** Each tab competes for the learner's attention. 2-4 tabs per challenge is ideal. If you need more, consider whether the challenge scope is too broad.
- **Wrong tab for cloud consoles.** AWS/Azure/GCP consoles require a full browser context (cookies, popups, redirects). A service tab will not work — use a virtual browser.

# Virtual Browser (Sandbox Service)

Full Chromium browser rendered as a tab in the Instruqt player, pointing at a web application running inside the sandbox. Use instead of a service tab when the application needs a real browser context -- JavaScript-heavy SPAs, apps that open popups, OAuth flows, or anything that breaks inside an iframe.

## When to Use

- The web application fails in a service tab iframe due to CSP restrictions that cannot be stripped, `window.top` checks, popup windows, or OAuth redirects.
- The exercise requires learners to interact with the browser's address bar, developer tools, or multiple browser tabs.
- The application is complex enough (heavy JavaScript, WebSocket connections, service workers) that iframe embedding is unreliable.

## Config.yml Example

The virtual browser is defined in the `virtualbrowsers:` section of `config.yml` and referenced by a `type: browser` tab in the assignment frontmatter.

```yaml
# config.yml
version: "3"
virtualmachines:
  - name: workstation
    image: instruqt/docker-28-3
    machine_type: n1-standard-2
    shell: /bin/bash
    allow_external_ingress:
      - http
      - https
      - high-ports
    nested_virtualization: true

virtualbrowsers:
  - name: app-browser
    url: http://workstation:8080
```

```yaml
# assignment.md frontmatter
tabs:
  - title: Terminal
    type: terminal
    hostname: workstation
  - title: Web App
    type: browser
    hostname: app-browser
```

## Pairs With

Any base compute pattern that runs a web server -- VMs with Docker, standalone VMs, containers. The `url:` in the virtual browser entry points to an internal sandbox hostname and port, so the application does not need to be externally accessible (no `allow_external_ingress` required for the virtual browser to reach it, though you still need it for service tabs on the same host).

## Common Patterns

- **Replacement for broken service tabs.** When a service tab renders blank or partially due to CSP/iframe issues that cannot be fixed with header overrides, switch to a virtual browser pointing at the same internal URL.
- **SPA development workshop.** React/Angular/Vue applications that use client-side routing, service workers, or WebSocket connections work reliably in a virtual browser but often break in an iframe.
- **Application with authentication flow.** Apps that redirect to a login page, use OAuth popups, or set cookies with `SameSite=Strict` need a real browser context.

## Common Pitfalls

- **Higher resource cost.** Each virtual browser runs a headless Chromium instance. This consumes more sandbox resources than a service tab (which is just an iframe). Do not use a virtual browser when a service tab works fine.
- **Missing virtualbrowsers entry.** The `type: browser` tab in the assignment references a `hostname:` that must match a `name:` in the `virtualbrowsers:` block of `config.yml`. A mismatch causes the tab to fail silently.
- **URL not reachable at startup.** If the web application is not ready when the virtual browser loads, it shows an error page. Use setup scripts to wait for the service before finishing.
- **Confusing sandbox virtual browser with external virtual browser.** A sandbox virtual browser points at an internal URL (`http://workstation:8080`). For external URLs (cloud consoles, SaaS products), use a virtual browser with an external URL instead.

## When NOT to Use

- **The application works in a service tab.** Service tabs are lighter and load faster. Only switch to a virtual browser when iframe embedding genuinely fails.
- **The target is an external URL.** For cloud consoles, SaaS portals, or any URL outside the sandbox, use a virtual browser with an external URL (see `virtual-browser-external.md`).
- **The learner only needs to view a static page.** If the web content is read-only documentation or a simple status page, a service tab or `type: website` tab is sufficient.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- Single-page application development with a React or Vue frontend
- Identity provider configuration where the app performs OAuth redirects
- Web application security testing requiring browser developer tools
- Complex dashboard application that uses popups and multi-tab navigation

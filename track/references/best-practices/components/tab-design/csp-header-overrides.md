# CSP and X-Frame-Options Header Overrides

Evaluates how Content-Security-Policy and X-Frame-Options headers are handled to allow web UIs to render inside the Instruqt player iframe.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | CSP/X-Frame-Options blocking is unaddressed; embedded UIs show blank frames or "refused to connect" errors |
| 2 | Below Standard | An approach is attempted but incorrectly implemented -- wrong header names, socat used instead of a real solution, or proxy misconfigured |
| 3 | Adequate | Headers are stripped and the UI loads, but the approach is overly complex for the situation (e.g., full nginx config when service tab headers would suffice) |
| 4 | Good | Embedded UIs load without CSP errors; the chosen approach works correctly (production baseline) |
| 5 | Excellent | Approach is chosen for minimal complexity -- service tab headers when possible, reverse proxy only when necessary, well-documented in setup script comments |

## Guidance

When a web UI sets `Content-Security-Policy`, `X-Frame-Options`, or related headers, the Instruqt player iframe cannot render it. Three approaches exist, in order of preference:

### Approach 1: Service tab custom headers (simplest)

Instruqt service tabs support `custom_response_headers` to strip or replace headers returned by the application. This requires no setup script changes.

```yaml
- title: Application UI
  type: service
  hostname: workstation
  port: 8080
  custom_response_headers:
    - "Content-Security-Policy: "
    - "X-Frame-Options: "
```

Setting a header to an empty value (note the trailing space after the colon) strips it from the response. You can also strip request headers that trigger CORS issues:

```yaml
  custom_request_headers:
    - "Origin: "
```

Use this approach when: the application is directly accessible on a port and only the response headers need modification.

### Approach 2: Caddy reverse proxy

When service tab headers are not sufficient (e.g., the application rewrites URLs, requires TLS termination, or has complex header interactions), use Caddy as a reverse proxy.

```
{
    auto_https off
}

:8888 {
    reverse_proxy localhost:8080 {
        header_down -Content-Security-Policy
        header_down -X-Frame-Options
        header_down -X-Content-Type-Options
    }
}
```

Key requirement: `auto_https off` must be set in the global options block. Without it, Caddy attempts ACME certificate provisioning inside the sandbox, which fails.

The service tab then points to the Caddy port:

```yaml
- title: Application UI
  type: service
  hostname: workstation
  port: 8888
```

Use this approach when: you need URL rewriting, TLS termination, or the application has headers that interact in ways service tab overrides cannot handle.

### Approach 3: nginx reverse proxy

Similar to Caddy but uses nginx. Useful when nginx is already present in the sandbox.

```nginx
server {
    listen 8888;

    location / {
        proxy_pass http://localhost:8080;
        proxy_hide_header Content-Security-Policy;
        proxy_hide_header X-Frame-Options;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Use this approach when: nginx is already installed and configured for other purposes in the sandbox.

### What does NOT work: socat

socat forwards raw TCP streams. It does not parse or modify HTTP headers. Using socat to forward ports does not strip CSP or X-Frame-Options headers:

```bash
# THIS DOES NOT WORK for CSP/X-Frame-Options
socat TCP-LISTEN:8888,fork TCP:localhost:8080
```

Good -- simplest approach, service tab headers:

```yaml
- title: ArgoCD
  type: service
  hostname: workstation
  port: 8080
  custom_response_headers:
    - "Content-Security-Policy: "
    - "X-Frame-Options: "
  custom_request_headers:
    - "Origin: "
```

Good -- Caddy proxy with auto_https off:

```
{
    auto_https off
}

:9090 {
    reverse_proxy localhost:8080 {
        header_down -Content-Security-Policy
        header_down -X-Frame-Options
    }
}
```

Bad -- socat for header stripping (does not work):

```bash
socat TCP-LISTEN:9090,fork TCP:localhost:8080
```

Bad -- Caddy without auto_https off (ACME fails in sandbox):

```
:9090 {
    reverse_proxy localhost:8080 {
        header_down -Content-Security-Policy
    }
}
```

Bad -- overly complex nginx config when service tab headers would suffice:

```nginx
# 20 lines of nginx config for something custom_response_headers handles in 2 lines
```

## What to Watch For

- Always try service tab `custom_response_headers` first -- it is zero-setup and handles the majority of cases
- The trailing space after the colon in `"X-Frame-Options: "` is intentional -- it sets the header to an empty value, effectively stripping it
- Caddy requires `auto_https off` in the global block; omitting it causes certificate provisioning failures that are hard to debug
- socat is a common incorrect suggestion -- it operates at the TCP level and cannot modify HTTP headers
- Some UIs set both `Content-Security-Policy` and `X-Frame-Options` -- strip both to be safe
- `custom_request_headers` stripping `Origin` can resolve CORS pre-flight failures that occur alongside CSP issues
- Test in the actual Instruqt player, not just in a direct browser tab -- CSP issues only manifest when the page is loaded inside an iframe
- If none of the three approaches work, `new_window: true` on the service tab is the escape hatch (opens in a separate browser tab)

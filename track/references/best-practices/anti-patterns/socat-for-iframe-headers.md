# Socat for Iframe Headers

Using `socat` to fix iframe-blocking HTTP headers like `X-Frame-Options` or `Content-Security-Policy`.

## Why It's Bad

socat operates at the TCP level and cannot inspect or modify HTTP headers. It forwards raw bytes between sockets. Attempts to strip `X-Frame-Options` or inject `Content-Security-Policy` headers with socat silently do nothing, leaving the iframe broken.

## What to Do Instead

Use a reverse proxy that understands HTTP:

- **Caddy**: Use `header_down` to strip or rewrite response headers.
- **nginx**: Use `proxy_hide_header` to remove unwanted headers.
- **Instruqt service tab**: Use `custom_request_headers` / `custom_response_headers` in config.yml.

socat is fine for pure TCP port-forwarding where no header manipulation is needed.

## Examples

Bad -- trying to strip X-Frame-Options with socat:

```bash
socat TCP-LISTEN:8080,fork,reuseaddr TCP:localhost:3000
# X-Frame-Options: DENY still present -- socat can't touch it
```

Good -- using Caddy to strip the header:

```
:8080 {
    reverse_proxy localhost:3000 {
        header_down -X-Frame-Options
        header_down -Content-Security-Policy
    }
}
```

Good -- using the Instruqt service tab config:

```yaml
tabs:
  - title: App
    type: service
    port: 3000
    custom_response_headers:
      X-Frame-Options: ""
```

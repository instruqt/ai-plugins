# GUI Access Method

Choosing between Guacamole RDP, Guacamole VNC, KasmVNC, and noVNC when a track needs graphical desktop or GUI application access.

## Comparison

| Aspect | Guacamole RDP | Guacamole VNC | KasmVNC | noVNC |
|--------|--------------|---------------|---------|-------|
| Protocol | RDP | VNC | VNC (built-in) | VNC |
| Architecture | VM + Guacamole container | VM + Guacamole container | VM + KasmVNC Docker container | VM only (no extra container) |
| Windows support | Yes (native) | No | No | No |
| Setup complexity | Medium (xrdp + Guacamole + user-mapping) | Medium (VNC server + Guacamole + user-mapping) | Low (single Docker container) | Low (VNC server + websockify) |
| Auto-resize | Yes (`resize-method: display-update`) | No (fixed resolution) | Yes | No (fixed resolution) |
| Resource overhead | ~512 MB (Guacamole container) | ~512 MB (Guacamole container) | ~2 GB (KasmVNC container) | Minimal |
| Image ecosystem | N/A | N/A | Application-specific images (Chrome, Firefox, desktop) | N/A |
| Best for | Windows, full Linux desktops | Headless Linux with VNC server | Specific GUI apps in containers | Quick prototyping, simple access |

## Decision Flow

```
Need Windows desktop?
  --> Guacamole RDP

Need a specific GUI app with a Docker image?
  --> KasmVNC (check kasmweb/* images)

Need full Linux desktop with polish (auto-resize, clipboard)?
  --> Guacamole RDP (install xrdp + XFCE)

Target system already runs VNC?
  --> Guacamole VNC

Quick prototype, minimal setup?
  --> noVNC

Teaching VNC/remote access concepts?
  --> noVNC (it IS the subject matter)
```

## Trade-offs

### Guacamole RDP
- **Pros:** Best UX (auto-resize, clipboard, multi-session), Windows support, proven at scale.
- **Cons:** Two setup scripts (VM + Guacamole container), user-mapping.xml boilerplate, heavier resource footprint.

### Guacamole VNC
- **Pros:** Works with existing VNC servers, same Guacamole UX.
- **Cons:** No auto-resize, clipboard varies by VNC server, single shared display.

### KasmVNC
- **Pros:** Single Docker container, application-specific images, no separate Guacamole needed, good for containerized GUI apps.
- **Cons:** Large images (1-2 GB pull time), UID 1000 permission quirks, `--shm-size=2g` required.

### noVNC
- **Pros:** Simplest setup (no extra container), minimal resource overhead.
- **Cons:** No auto-resize, VNC password visible in URL, websockify must stay running, least polished UX.

## Common Mistakes

- **Using KasmVNC for a full desktop when Guacamole RDP would be better.** KasmVNC excels at single-app delivery. For full desktop workflows with clipboard and resize, Guacamole RDP provides a better experience.
- **Using Guacamole when KasmVNC would be simpler.** If you just need one GUI app (an IDE, a browser), KasmVNC avoids the Guacamole container + user-mapping complexity.
- **Forgetting to poll for readiness.** All four methods require the GUI service to be ready before the tab loads. Always poll in setup scripts.
- **Under-sizing the VM.** All GUI methods need at least `n1-standard-4`. Budget more for heavyweight desktops (GNOME, KDE) or resource-intensive applications.

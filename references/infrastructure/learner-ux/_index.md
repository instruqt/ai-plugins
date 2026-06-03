# Learner UX Components

Infrastructure building blocks that define how learners interact with the sandbox. Each track composes one or more of these components into its tab layout.

## Native

Built into Instruqt. Configured in `config.yml` and assignment frontmatter only -- no additional software to install.

- [Terminal Tab](native/terminal-tab.md) -- Shell access to a sandbox host. The default interface; every track has at least one.
- [Service Tab](native/service-tab.md) -- Iframe proxy to a web UI running inside the sandbox (hostname + port).
- [Editor Tab](native/editor-tab.md) -- In-browser Monaco code editor for files or directories on a sandbox host.
- [Virtual Browser (Sandbox)](native/virtual-browser-sandbox.md) -- Full Chromium browser pointing at an internal sandbox URL. Use when service tabs break due to iframe restrictions.
- [Virtual Browser (External)](native/virtual-browser-external.md) -- Full Chromium browser pointing at an external URL (cloud console, SaaS portal).

## Installed

Software installed on a VM and exposed via a service tab or virtual browser. Requires a VM base compute pattern and setup/image configuration.

- Code-Server -- Full VS Code in the browser. Use when learners need extensions, integrated terminal, or Git integration beyond what the native editor provides.
- Guacamole RDP -- Apache Guacamole providing remote desktop access to a Windows or Linux GUI. Use for desktop application workshops.
- Guacamole VNC -- Apache Guacamole providing VNC access. Lighter than RDP; use for Linux desktop environments.
- KasmVNC -- High-performance VNC with a web client. Use for GPU-accelerated or multimedia desktop sessions.
- noVNC -- Lightweight browser-based VNC client. Use for simple Linux desktop access without Guacamole's overhead.

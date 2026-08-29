---
topic: antigravity-docker
category: project
tags: [project, antigravity-docker]
updated_at: 2026-08-29T11:43:05.133220+00:00
confidence: 0.95
---

# Project: Antigravity-Docker

- Replaced /var/run/docker.sock mounting with an SSH-based web terminal gateway
(ttyd) to execute host commands securely without container socket exposure.
- Integrated VS Code Web IDE (code-server) and Host Terminal (ttyd) via
auth-proxy sidebar injection, toggleable with ENABLE_IDE and ENABLE_TERMINAL
environment variables.

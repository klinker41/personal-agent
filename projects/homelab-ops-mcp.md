---
topic: homelab-ops-mcp
category: project
tags: [project, homelab-ops-mcp]
updated_at: 2026-10-15T00:01:11.186380+00:00
confidence: 0.95
---

# Project: Homelab-Ops-Mcp

- Decoupled from Antigravity as a standalone project (`/workspace/homelab-ops`)
backed by `git@github.com:klinker41/homelab-ops-mcp.git` .
- Runs as a dedicated Docker container on the host with explicit environment
variables/config and registers into the Antigravity plugin via root
`mcp_config.json` .
- Includes tools for Unraid GraphQL notification retrieval and dismissal, server
health checks, and media stack searching and watch-status marking.

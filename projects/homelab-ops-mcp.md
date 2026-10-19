---
topic: homelab-ops-mcp
category: project
tags: [project, homelab-ops-mcp]
updated_at: 2026-10-19T06:03:44.598485+00:00
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

- Uptime Kuma's 'AIbox Notifications' monitor tracks unread notification counts
on Unraid; transient alerts (such as temporary Docker disk utilization spikes)
keep the monitor in a DOWN state until unread notifications are cleared via
unraid_dismiss_notification.

- Provides 'jellyfin_get_podcast_episodes' and 'jellyfin_update_podcast_episode'
tools to query unformatted podcast episodes and update their titles, disc
numbers, and track numbers.

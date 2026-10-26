---
topic: uptime-kuma-diagnostics
category: knowledge
tags: [knowledge, uptime-kuma-diagnostics]
updated_at: 2026-10-26T06:01:33.103114+00:00
confidence: 0.95
---

# Knowledge: Uptime-Kuma-Diagnostics

- Because Uptime Kuma (`uptime.klinker.computer`) monitors services from the
primary site (`unRAID AIbox`), timeout alerts for Cabin-hosted services (`unRAID
NAS` / `unRAID Seedbox` ) can be caused by cross-site WAN or `gluetun-home`
WireGuard tunnel outages even when the local container and
`*.klinker-cabin.computer` ingress are healthy.

- Because the Uptime Kuma instance (`uptime.klinker.computer`) probes
`.klinker-cabin.computer` endpoints across an inter-site network link, transient
inter-site connectivity drops cause 48,000ms probe timeout alerts for
`.klinker-cabin.computer` services even when the local containers and endpoints
are healthy.

- Uptime Kuma API keys (`UPTIME_KUMA_API_KEY`) only authenticate the read-only
Prometheus `/metrics` endpoint; mutating monitors or status pages
(`deleteMonitor`, `saveStatusPage` ) requires authenticating via Uptime Kuma's
Socket.IO WebSocket with an admin username and password.

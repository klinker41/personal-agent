---
topic: unraid-notification-monitoring
category: knowledge
tags: [knowledge, unraid-notification-monitoring]
updated_at: 2026-10-23T06:04:21.118852+00:00
confidence: 0.95
---

# Knowledge: Unraid-Notification-Monitoring

- Uptime Kuma monitors tracking Unraid notification states remain DOWN even
after an underlying transient issue self-resolves because Unraid keeps notices
in unread status until they are explicitly dismissed.

- Because Uptime Kuma (`uptime.klinker.computer`) is hosted at the remote site
and probes Cabin services (`*.klinker-cabin.computer`) over WAN, remote WAN or
WireGuard outages trigger false-positive `timeout of 48000ms exceeded` Down
alerts across Cabin monitors while `kuma_get_monitors` itself times out (`HTTP
0`).

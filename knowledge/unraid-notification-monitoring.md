---
topic: unraid-notification-monitoring
category: knowledge
tags: [knowledge, unraid-notification-monitoring]
updated_at: 2026-10-28T06:01:59.242453+00:00
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

- During Cabin utility power outages, `unraid-seedbox` shuts down after ~8–9
minutes on UPS battery (`unraid-server`) to preserve battery capacity for
`unraid-nas` and networking equipment; checking `unraid-nas` notifications for
an active `UPS Alert` (`Power failure on UPS unraid-server. Running on
batteries.`) distinguishes a graceful UPS shutdown of `unraid-seedbox` from a
host or network crash.

- At the Cabin site, `unraid-nas` and `unraid-seedbox` share UPS status via
`apcupsd` ; during a utility power outage, `seedbox` is configured with a
~10-minute time-on-battery threshold to perform an early graceful shutdown and
conserve UPS battery runtime for `nas` and core network equipment.

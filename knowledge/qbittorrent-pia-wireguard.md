---
topic: qbittorrent-pia-wireguard
category: knowledge
tags: [knowledge, qbittorrent-pia-wireguard]
updated_at: 2026-11-24T07:20:48.620976+00:00
confidence: 0.95
---

# Knowledge: Qbittorrent-Pia-Wireguard

- In `binhex/arch-qbittorrentvpn` with `STRICT_PORT_FORWARD=yes`, if the
configured PIA WireGuard endpoint in `wireguard/wg0.conf` is removed from PIA's
port-forwarding-enabled server list (e.g., `nl-amsterdam.privacy.network:1337`),
port assignment fails and `watchdog.sh` waits indefinitely without starting
`qbittorrent-nox`, causing `TCP_CONNECTION_REFUSED` on the WebUI port (`8081`)
and `HTTP 502` at the reverse proxy while the container stays running; switching
`Endpoint` to an active port-forwarding hostname such as
`netherlands.pvt.site:1337` restores startup.

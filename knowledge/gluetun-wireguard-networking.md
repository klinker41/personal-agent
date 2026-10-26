---
topic: gluetun-wireguard-networking
category: knowledge
tags: [knowledge, gluetun-wireguard-networking]
updated_at: 2026-10-26T06:03:17.122601+00:00
confidence: 0.95
---

# Knowledge: Gluetun-Wireguard-Networking

- Gluetun's custom WireGuard parser strictly requires a raw IP address (not a
DDNS hostname or domain name) for the `Endpoint` field because it configures
iptables firewall killswitch rules for the endpoint IP prior to starting DNS or
bringing up the WireGuard interface.
- When a parent container (such as Gluetun) is recreated, any dependent
containers sharing its network stack via `network_mode: service:<container>`
remain bound to the destroyed network namespace and must be explicitly restarted
to regain connectivity.

- When the `gluetun-home` WireGuard container on `unraid-nas` restarts,
dependent containers sharing its network namespace (such as `semaphore-boulder`
) cause reverse proxy `502 Bad Gateway` errors until those dependent containers
are restarted to attach to the new network namespace.

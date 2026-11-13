---
topic: gluetun-wireguard-networking
category: knowledge
tags: [knowledge, gluetun-wireguard-networking]
updated_at: 2026-11-13T07:32:35.187917+00:00
compacted_at: 2026-11-13T07:32:35.187917+00:00
confidence: 0.95
---

# Knowledge: Gluetun-Wireguard-Networking

- Gluetun's custom WireGuard parser strictly requires a raw IP address (not a
  DDNS hostname or domain name) for the `Endpoint` field because it configures
  iptables firewall killswitch rules for the endpoint IP before starting DNS or
  bringing up the WireGuard interface.
- Hosts on the remote Boulder LAN (such as `pi-studio` at `192.168.1.72`) are
  reachable via SSH from the Cabin network by hopping through the
  `semaphore-boulder` container attached to the `gluetun-home` WireGuard
  network namespace on `unraid-nas`.
- When a Gluetun container (`qmcgaw/gluetun`, such as `gluetun-home` or
  `gluetun-home-backup` on `unraid-nas`) restarts in place, dependent
  containers sharing its network namespace (e.g., `semaphore-boulder`) lose
  connectivity and cause upstream `502 Bad Gateway` errors until restarted.
- Docker resolves `network_mode: service:<service>`,
  `network_mode: container:<name>`, and `--net=container:<name>` to the target
  container's exact ID at creation time. When Gluetun is recreated with a new
  ID (e.g., during nightly/scheduled image updates or Komodo auto-updates),
  dependent sidecars remain pinned to the stale `container:<old_id>` namespace
  (causing exit code 143 and HTTP `502 Bad Gateway` errors on exposed ports).
  Standard restarts (`docker restart` or `komodo_restart_container`) fail with
  `joining network namespace of container: No such container: <old_id>`;
  dependents must be force-recreated (e.g., `docker compose up -d <service>`)
  to bind to the new Gluetun container ID.

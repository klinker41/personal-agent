---
topic: gluetun-wireguard-networking
category: knowledge
tags: [knowledge, gluetun-wireguard-networking]
updated_at: 2026-11-09T07:02:53.517553+00:00
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
dependent containers sharing its network namespace (such as `semaphore-boulder`)
cause reverse proxy `502 Bad Gateway` errors until those dependent containers
are restarted to attach to the new network namespace.

- Hosts on the remote Boulder LAN (such as `pi-studio` at `192.168.1.72`) can be
reached via SSH from the Cabin network by hopping through the
`semaphore-boulder` container attached to the `gluetun-home` WireGuard network
namespace on `unraid-nas`.

- When a Docker container uses `--net=container:<name>` (such as attaching to a
Gluetun container), Docker binds it to the target container's exact container ID
at creation time; if the Gluetun container is recreated with a new ID, a
standard `docker restart` (or `komodo_restart_container` ) on the dependent
container fails with
`joining network namespace of container: No such container: <old_id>` ,
requiring the dependent container to be recreated rather than just restarted.

- Docker pins `network_mode: container:<name>` to the target container's ID at
creation time; when a Gluetun container is recreated (such as during nightly
image updates), dependent containers lose network connectivity (causing upstream
502 Bad Gateway errors) and fail simple restarts with `joining network namespace
of container: No such container: <old-id>`, requiring a full container
recreation/redeploy to bind to the new Gluetun container ID.

- Docker containers configured with `network_mode: service:<gluetun>` exit with
code 143 (`joining network namespace of container: No such container: <old_id>`)
when the parent Gluetun container is recreated with a new ID, requiring the
dependent container to be recreated to attach to the new network namespace.

- When a Gluetun VPN container (`gluetun-home` or `gluetun-home-backup`) is
recreated during scheduled image updates, dependent containers using
`network_mode: container:<gluetun>` remain bound to the old deleted container
ID's network namespace (causing `502 Bad Gateway` errors) and must be recreated
to reattach to the new namespace.

- When a VPN container (such as `gluetun`) is recreated with a new container ID,
dependent containers configured with `network_mode: service:<vpn-service>` fail
with `joining network namespace of container: No such container:
<old-id>` because Docker resolves `service: ` network namespaces to a container
ID at creation time. A standard container restart cannot rebind the namespace;
the dependent container must be recreated via `docker compose up -d`.

- When Komodo's auto-updater recreates a Gluetun container (`gluetun-home` or
`gluetun-home-backup`), dependent containers configured with
`network_mode: service:<gluetun>` remain bound to the deleted container's
network namespace ID (`No such container: <id>`), resulting in `502 Bad Gateway`
errors until the dependent containers are force-recreated to attach to the new
Gluetun container ID.

---
topic: komodo-container-management
category: knowledge
tags: [knowledge, komodo-container-management]
updated_at: 2026-11-14T07:33:14.782339+00:00
compacted_at: 2026-11-14T07:33:14.782339+00:00
confidence: 0.95
---

# Knowledge: Komodo-Container-Management

- On GitHub Container Registry (`ghcr.io/moghtech/komodo-core` and
  `komodo-periphery`), `:latest` is frozen on Komodo v1 to avoid breaking
  auto-updaters; use the `:2` rolling tag for Komodo v2 (required for native
  `[[swarm]]` orchestration, outbound WebSocket periphery connections, and
  public/private key auth). Swarm stacks set `swarm = "<swarm_name>"`
  (connected to Swarm manager `Server` nodes running `komodo-periphery`)
  instead of `server = "<server_name>"`.
- Programmatic stack redeployment uses `POST /execute` with `X-Api-Key` and
  `X-Api-Secret` headers and JSON body
  `{"type": "DeployStack", "params": {"stack": "<stack_name>"}}`.
- Komodo Core provides built-in daily database backups
  (`BackupCoreDatabase` procedure), declarative TOML sync (`ResourceSync`),
  and native image polling (`poll_for_updates` / `auto_update`), replacing
  Watchtower, Shepherd, and external backup containers. When
  `auto_update = true` in `homelab.toml`, pushing to `homelab-stacks` pulls
  the remote registry image, overwriting any unpushed local builds.
- On resource-constrained hosts, pass `--quiet --wiredTigerCacheSizeGB 0.25`
  to Komodo's `mongo` container to cap WiredTiger cache memory around 250 MB.
- Komodo's native Slack alerter (`slack_client_rs`) sends only `text` and
  `blocks` without `username` or `icon_url` overrides (which Slack App-level
  Incoming Webhooks ignore anyway), requiring a dedicated Slack App webhook
  to customize the sender name and icon.
- Syncing `Swarm`, `Stack`, and `Alerter` definitions (`ResourceSync` /
  `CommitSync`) while target servers are offline/disabled (`Cannot send
  command when Server is unreachable or disabled`) or Swarm managers are
  disconnected (`Failed to request from Swarm: No managers connected`) marks
  the Swarm unhealthy, fires critical alerts, and fails the overall sync
  procedure even if online stacks deploy cleanly.
- In procedures, `RunSync` loops until pending resource changes converge;
  persistent diffs or container state mismatches fail with
  `procedure sync loop exited after max iterations`.
- If `komodo-periphery` running as a Docker Swarm service reports down while
  the host remains reachable via SSH/Ansible, check `docker info` and
  `journalctl -u docker` on the manager node for Swarm certificate
  expiration or `Swarm: error` states blocking service startup.
- Transient state and alert anomalies:
  - **Health checks & crash loops**: Containers in `(health: starting)`
    during redeploy/restart (e.g., `homepage` in `nas-dashboard`) may
    briefly flag a stack `unhealthy`, whereas ~15-second crash loops flap
    the parent stack between `Running` and `unhealthy`.
  - **Git poll resets**: Scheduled polls (`KOMODO_RESOURCE_POLL_INTERVAL`)
    run concurrent SSH `git pull` operations across stacks and
    ResourceSyncs; transient connection resets falsely flip stack state to
    `down` until the next poll cycle while containers stay healthy.
  - **03:00 nightly events**: Stacks with `auto_update = true` emit
    transient `running -> Down` alerts at 03:00 local time when the daily
    Global Auto Update pulls images and recreates containers. Separately,
    `pi-studio` `ServerUnreachable` alerts around 03:01–03:04 MST (10:00
    UTC) stem from recurring ~2-minute Boulder site WAN/router maintenance
    blips and self-recover once DNS and routing restore.
  - **Inbound WebSocket reconnects**: Servers with `address: ""` rely on
    `komodo-periphery` connecting inbound to `komodo-core`; after a
    `komodo-core` restart, if the first 15-second health check runs before
    periphery's reconnect backoff completes, a transient
    `ServerUnreachable` alert fires and self-resolves on the next poll.
  - **Jenkins builds on `unraid-nas`**: Transient 100% CPU alerts alongside
    ephemeral random-named `sha256:...` containers with `--log-driver=none`
    indicate active `docker build` jobs triggered by `jenkins-docker-agent`
    via the mounted host Docker socket.

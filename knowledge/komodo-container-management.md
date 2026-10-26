---
topic: komodo-container-management
category: knowledge
tags: [knowledge, komodo-container-management]
updated_at: 2026-10-26T06:31:56.366481+00:00
compacted_at: 2026-10-26T06:31:56.366481+00:00
confidence: 0.95
---

# Knowledge: Komodo-Container-Management

- On GitHub Container Registry (`ghcr.io/moghtech/komodo-core` and
  `komodo-periphery`), `:latest` is frozen on Komodo v1 to prevent breaking
  auto-updaters; use the `:2` rolling tag to deploy Komodo v2 (required for
  native `[[swarm]]` support, outbound WebSocket periphery connections, and
  public/private key auth).
- Komodo v2 supports native Docker Swarm orchestration via `[[swarm]]`
  resources connected to Swarm manager `Server` nodes running
  `komodo-periphery`; Swarm stacks reference `swarm = "<swarm_name>"` instead
  of `server = "<server_name>"`.
- Programmatic stack redeployment uses a single `POST /execute` request with
  `X-Api-Key` and `X-Api-Secret` headers and body
  `{"type": "DeployStack", "params": {"stack": "<stack_name>"}}`.
- Komodo Core includes built-in daily database backups (`BackupCoreDatabase`
  procedure), declarative TOML synchronization (`ResourceSync`), and native
  image update polling (`poll_for_updates` / `auto_update`) that replaces
  Watchtower, Shepherd, and external backup containers.
- On resource-constrained hosts, pass `--quiet --wiredTigerCacheSizeGB 0.25` to
  Komodo's native `mongo` container to cap WiredTiger cache memory around
  250 MB.
- Syncing or importing `Swarm`, `Stack`, and `Alerter` definitions
  (`ResourceSync` / `CommitSync`) when target servers are offline/disabled
  (`Cannot send command when Server is unreachable or disabled`) or Swarm
  managers are disconnected (`Failed to request from Swarm: No managers
  connected`) immediately marks the Swarm unhealthy, fires critical alerts, and
  fails the overall sync procedure even if online stacks deploy cleanly.
- In Komodo procedures, `RunSync` executes an iterative sync loop until pending
  resource changes converge; if a resource diff or container state mismatch
  persists across passes, the procedure fails with
  `procedure sync loop exited after max iterations`.
- If a Komodo server running `komodo-periphery` as a Docker Swarm service
  reports down while the host remains reachable via SSH/Ansible, check
  `docker info` and `journalctl -u docker` on the manager node for Swarm
  certificate expiration or `Swarm: error` states preventing service startup.
- Because Komodo polls container states periodically, containers in a
  `(health: starting)` window during redeploy/restart (e.g., `homepage` in
  `nas-dashboard`) may briefly flag the stack as `unhealthy`, while containers
  in a ~15-second crash loop cause the parent stack to flap between `Running`
  and `unhealthy`.
- Transient 100% CPU alerts on `unraid-nas` accompanied by ephemeral
  random-named containers using `sha256:...` images and `--log-driver=none`
  typically indicate an active `docker build` triggered by
  `jenkins-docker-agent` via the mounted host Docker socket.
- Komodo's native Slack alerter (`slack_client_rs`) only sends `text` and
  `blocks` fields without `username` or `icon_url` overrides (which Slack
  App-level Incoming Webhooks ignore anyway), requiring a dedicated Slack App
  webhook to customize the sender name and icon.

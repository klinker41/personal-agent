---
topic: komodo-container-management
category: knowledge
tags: [knowledge, komodo-container-management]
updated_at: 2026-11-19T07:32:01.938045+00:00
compacted_at: 2026-11-19T07:32:01.938045+00:00
confidence: 0.95
---

# Knowledge: Komodo-Container-Management

- **Images & Swarm v2**: On GHCR (`ghcr.io/moghtech/komodo-core` and
  `komodo-periphery`), `:latest` is frozen on Komodo v1 to avoid breaking
  auto-updaters; use the `:2` rolling tag for v2 (required for native
  `[[swarm]]` orchestration, outbound WebSocket periphery connections, and
  public/private key auth). Swarm stacks set `swarm = "<swarm_name>"` (via
  Swarm manager `Server` nodes running `komodo-periphery`) instead of
  `server = "<server_name>"`.
- **API Redeployment**: Trigger stack deploys via `POST /execute` with
  `X-Api-Key` and `X-Api-Secret` headers and JSON body
  `{"type": "DeployStack", "params": {"stack": "<stack_name>"}}`.
- **Built-in Automation**: Komodo Core provides daily DB backups
  (`BackupCoreDatabase` procedure), declarative TOML sync (`ResourceSync`),
  and native image polling (`poll_for_updates` / `auto_update`), replacing
  Watchtower, Shepherd, and external backup containers. When
  `auto_update = true` in `homelab.toml`, pushing to `homelab-stacks` pulls
  the remote registry image, overwriting unpushed local builds.
- **MongoDB Memory Cap**: On resource-constrained hosts, pass
  `--quiet --wiredTigerCacheSizeGB 0.25` to Komodo's `mongo` container to cap
  WiredTiger cache around 250 MB.
- **Slack Alerter**: Native `slack_client_rs` sends only `text` and `blocks`
  without `username` or `icon_url` overrides (ignored by Slack App Incoming
  Webhooks anyway); use a dedicated Slack App webhook to customize sender
  name and icon.
- **Sync, Deploy & Procedure Failures**:
  - **Offline targets**: Syncing `Swarm`, `Stack`, and `Alerter` definitions
    (`ResourceSync` / `CommitSync`) while target servers are offline/disabled
    (`Cannot send command when Server is unreachable or disabled`) or Swarm
    managers are disconnected (`Failed to request from Swarm: No managers
    connected`) marks the Swarm unhealthy, fires critical alerts, and fails
    the sync procedure even if online stacks deploy cleanly.
  - **Sync convergence**: In procedures, `RunSync` loops until pending
    changes converge; persistent diffs or state mismatches fail with
    `procedure sync loop exited after max iterations`.
  - **Registry rate limits**: Transient `toomanyrequests` errors on `lscr.io`
    or GHCR during `DeployStack` cause `Compose Pull` to exit non-zero and
    skip `Compose Up`; re-running the procedure via `RunProcedure` resolves
    deployment once the rate limit clears.
- **Swarm Periphery Diagnostics**: If `komodo-periphery` running as a Docker
  Swarm service reports down while the host is reachable via SSH/Ansible,
  check `docker info` and `journalctl -u docker` on the manager node for
  Swarm certificate expiration or `Swarm: error` states blocking startup.
- **Transient State & Alert Anomalies**:
  - **Health checks & crash loops**: Containers in `(health: starting)` during
    redeploy/restart (e.g., `homepage` in `nas-dashboard`) briefly flag a
    stack `unhealthy`; ~15-second crash loops flap the parent stack between
    `Running` and `unhealthy`.
  - **Git poll resets**: Scheduled polls (`KOMODO_RESOURCE_POLL_INTERVAL`) run
    concurrent SSH `git pull` operations across stacks and ResourceSyncs;
    transient connection resets falsely flip stack state to `down` until the
    next poll cycle while containers stay healthy.
  - **Nightly auto-updates (03:00)**: Stacks with `auto_update = true` emit
    transient `running -> Down` alerts at 03:00 local time when Global Auto
    Update pulls images and recreates containers.
  - **Periphery & `pi-studio` reconnects**: Servers with `address: ""` rely on
    inbound `komodo-periphery` WebSockets; after `komodo-core` restarts, if
    the initial 15-second health check runs before reconnect backoff finishes,
    a transient `ServerUnreachable` alert fires and self-resolves next poll.
    Remote `pi-studio` also fires transient `ServerUnreachable` alerts during
    `unraid-nas` reboots (when `komodo-core` starts before `gluetun-home`
    establishes the Boulder WireGuard tunnel) and around 03:01–03:04 MST
    (10:00 UTC) during ~2-minute Boulder WAN/router maintenance blips,
    self-recovering once tunnels, DNS, and routing restore.
  - **Jenkins builds on `unraid-nas`**: Transient 100% CPU alerts alongside
    ephemeral random-named `sha256:...` containers (`--log-driver=none`)
    indicate active `docker build` jobs from `jenkins-docker-agent` via the
    mounted host Docker socket.

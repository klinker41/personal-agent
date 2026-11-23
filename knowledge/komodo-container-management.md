---
topic: komodo-container-management
category: knowledge
tags: [knowledge, komodo-container-management]
updated_at: 2026-11-23T07:05:40.403596+00:00
compacted_at: 2026-11-21T07:40:45.437726+00:00
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
- **Core Hosting & MongoDB**: `komodo-core` (`komodo.klinker-cabin.computer`)
  runs on `unraid-nas` alongside `komodo-mongo`, `gluetun-home`, and
  `nginx-proxy-manager`. On resource-constrained hosts, pass
  `--quiet --wiredTigerCacheSizeGB 0.25` to `mongo` to cap WiredTiger cache
  around 250 MB.
- **API & MCP Tooling**:
  - **Redeploy API**: `POST /execute` with `X-Api-Key` and `X-Api-Secret`
    headers and JSON body
    `{"type": "DeployStack", "params": {"stack": "<stack_name>"}}`.
  - **`komodo_get_container_logs`**: LLM callers often pass `lines`,
    `limit`, `num_lines`, or `line_count` alongside boolean `tail: true`;
    normalize these aliases to integer `tail` to avoid unexpected keyword
    argument errors.
- **Built-in Automation**: Komodo Core provides daily DB backups
  (`BackupCoreDatabase` procedure), declarative TOML sync (`ResourceSync`),
  and native image polling (`poll_for_updates` / `auto_update`), replacing
  Watchtower, Shepherd, and external backup containers. With
  `auto_update = true` in `homelab.toml`, pushing to `homelab-stacks` pulls
  the remote registry image, overwriting unpushed local builds.
- **Slack Alerter**: Native `slack_client_rs` sends only `text` and `blocks`
  without `username` or `icon_url` overrides (ignored by Slack App Incoming
  Webhooks anyway); use a dedicated Slack App webhook to customize sender
  name and icon.
- **Sync, Deploy & Procedure Failures**:
  - **Offline targets**: Syncing `Swarm`, `Stack`, and `Alerter` definitions
    (`ResourceSync` / `CommitSync`) while servers are offline/disabled
    (`Cannot send command when Server is unreachable or disabled`) or Swarm
    managers are disconnected (`Failed to request from Swarm: No managers
    connected`) marks the Swarm unhealthy, fires critical alerts, and fails
    the sync procedure even if online stacks deploy cleanly.
  - **Sync convergence**: `RunSync` loops until pending changes converge;
    persistent diffs or state mismatches fail with
    `procedure sync loop exited after max iterations`.
  - **Registry rate limits**: Transient `toomanyrequests` errors on `lscr.io`
    or GHCR during `DeployStack` cause `Compose Pull` to exit non-zero and
    skip `Compose Up`; re-running via `RunProcedure` resolves deployment once
    the rate limit clears.
- **Periphery Connectivity & Diagnostics**:
  - **Swarm service down**: If `komodo-periphery` running as a Docker Swarm
    service reports down while the host responds over SSH/Ansible, check
    `docker info` and `journalctl -u docker` on the manager node for Swarm
    certificate expiration or `Swarm: error` states blocking startup.
  - **Stalled WebSocket reconnects**: Remote outbound WebSocket
    `komodo-periphery` containers (`address: ""` on Core) can stall after a
    `komodo-core` restart or `unraid-nas` reboot with `WARN Timed out
    waiting for Ping: deadline has elapsed` without reconnecting; run
    `docker restart komodo-periphery` on the affected host to restore
    connectivity immediately.
- **Transient State & Alert Anomalies**:
  - **Periphery & `pi-studio` reconnects**: After `komodo-core` restarts or
    `unraid-nas` reboots, Core's initial 15-second health check can run
    before WebSocket reconnect backoff completes or before `gluetun-home`
    establishes WireGuard tunnels (e.g., to Boulder `pi-studio`), firing
    transient `ServerUnreachable` / `unreachable` alerts that self-resolve
    next poll (unless stalled on Ping timeout above). `pi-studio` also
    fires transient `ServerUnreachable` alerts around 03:01–03:04 MST
    (10:00 UTC) during ~2-minute Boulder WAN/router maintenance blips,
    self-recovering once tunnels, DNS, and routing restore.
  - **Health checks & crash loops**: Containers in `(health: starting)`
    during redeploy/restart (e.g., `homepage` in `nas-dashboard`) briefly
    flag a stack `unhealthy`; ~15-second crash loops flap the parent stack
    between `Running` and `unhealthy`.
  - **Git poll resets**: Scheduled polls (`KOMODO_RESOURCE_POLL_INTERVAL`)
    run concurrent SSH `git pull` operations across stacks and
    ResourceSyncs; transient connection resets falsely flip stack state to
    `down` until the next poll cycle while containers stay healthy.
  - **Nightly auto-updates (03:00)**: Stacks with `auto_update = true` emit
    transient `running -> Down` alerts at 03:00 local time when Global Auto
    Update pulls images and recreates containers.
  - **Jenkins builds on `unraid-nas`**: Transient 100% CPU alerts alongside
    ephemeral random-named `sha256:...` containers (`--log-driver=none`)
    indicate active `docker build` jobs from `jenkins-docker-agent` via the
    mounted host Docker socket.

- When `komodo-periphery`'s outbound WebSocket to Komodo Core stalls with `WARN
Timed out waiting for Ping: deadline has elapsed`, the process stays alive and
lacks a default Docker healthcheck, so `RestartPolicy=unless-stopped` does not
recover it and `docker restart komodo-periphery` is required.
- `komodo-periphery` must not self-update via Komodo's `GlobalAutoUpdate` or
`DeployStack` (exclude it with the `komodo.skip` label), because
`komodo-periphery` executes `docker compose up -d` itself and stopping its own
container mid-command aborts recreation and leaves the host offline.
- Use `nickfedor/watchtower` (maintained fork supporting Docker Engine 29.x)
scoped explicitly to `komodo-core` and `komodo-periphery` container names to
safely auto-update Komodo containers externally.

- Leaving `alert_types` empty (`[]`) on a Komodo alerter forwards all alert
categories, including routine informational events (`StackAutoUpdated`,
`DeploymentAutoUpdated`, `StackImageUpdateAvailable`,
`DeploymentImageUpdateAvailable`, `ResourceSyncPendingUpdates`, `ScheduleRun`,
and `None`).
- To restrict a Komodo alerter to failure, health, and manual test
notifications, explicitly set `alert_types` to `ServerUnreachable`, `ServerCpu`,
`ServerMem`, `ServerDisk`, `ServerVersionMismatch`, `SwarmUnhealthy`,
`AwsBuilderTerminationFailed`, `ContainerStateChange`, `StackStateChange`,
`ProcedureFailed`, `ActionFailed`, `BuildFailed`, `RepoBuildFailed`, `Test`, and
`Custom`.

- Komodo v2 lacks a built-in alert duration threshold (`moghtech/komodo#1384`);
`AlertBuffer::ready_to_open` triggers `ServerCpu` alerts after 2 consecutive
monitoring polls above threshold, making spike sensitivity directly tied to
`KOMODO_MONITORING_INTERVAL` unless debounced externally via a Custom Alerter
webhook.

- Per-server CPU alerts in Komodo are controlled via `send_cpu_alerts`
(alongside `cpu_warning` and `cpu_critical` percentage thresholds) under
`[server.config]` in Resource Sync TOML files or via the `UpdateServer` API.
- Komodo Resource Syncs using `match_tags` only reconcile and overwrite
resources carrying the matching tags, leaving untagged resources (such as
standalone alerters) untouched in Komodo's database.

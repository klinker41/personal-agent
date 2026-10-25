---
topic: komodo-container-management
category: knowledge
tags: [knowledge, komodo-container-management]
updated_at: 2026-10-25T06:01:49.213802+00:00
confidence: 0.95
---

# Knowledge: Komodo-Container-Management

- On GitHub Container Registry (`ghcr.io/moghtech/komodo-core` and
`komodo-periphery` ), the `:latest` image tag is frozen on Komodo v1 to prevent
breaking v1 installations using auto-updaters; use the `:2` rolling tag to
deploy Komodo v2 (required for native `[[swarm]]` support, outbound WebSocket
periphery connections, and public/private key auth).
- Komodo v2 supports native Docker Swarm orchestration via `[[swarm]]` resources
connected to Swarm manager `Server` nodes running `komodo-periphery` ; Swarm
stacks reference `swarm = "<swarm_name>"` instead of `server = "<server_name>"`
.
- Programmatic stack redeployment in Komodo uses a single `POST /execute`
request with `X-Api-Key` and `X-Api-Secret` headers and body
`{"type": "DeployStack", "params": {"stack": "<stack_name>"}}` .
- Komodo Core includes built-in daily database backups (`BackupCoreDatabase`
procedure) and declarative TOML synchronization (`ResourceSync`) for backing up
Server, Swarm, and Stack configurations without external backup containers, as
well as native image update polling (`poll_for_updates` / `auto_update` ) that
replaces Watchtower and Shepherd.
- When deploying Komodo with its native MongoDB backend on resource-constrained
hosts, pass `--quiet --wiredTigerCacheSizeGB 0.25` to the `mongo` container to
cap WiredTiger cache memory around 250 MB.

- When syncing Swarm and Stack resources in Komodo (`CreateSwarm` /
`CreateStack` ), `komodo-core` immediately marks the Swarm resource as unhealthy
(`Failed to request from Swarm: No managers connected`) and triggers alerts if
no `komodo-periphery` agent is actively connected as a Swarm manager for that
cluster.

- Executing a Komodo Resource Sync (`CommitSync`) that imports `Swarm` , `Stack`
, and `Alerter` definitions before the swarm's manager nodes have connected to
`komodo-core` results in `Failed to request from Swarm: No managers connected`
warnings and immediately fires a critical `Swarm <name> is unhealthy` alert.

- Komodo resource syncs (`ResourceSync` / `gitops-deploy` ) report an overall
procedure failure when stacks target offline or disabled servers (`Cannot send
command when Server is unreachable or disabled`) or disconnected swarms (`No
managers connected`), even when stacks targeting online hosts deploy cleanly.

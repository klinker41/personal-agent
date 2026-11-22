---
topic: homelab-stacks
category: projects
tags: [komodo, gitops, docker, stacks, homelab]
updated_at: 2026-11-22T07:21:12.557743+00:00
compacted_at: 2026-11-20T07:32:12.326888+00:00
confidence: 1.0
---

# Homelab-Stacks

## GitOps & Komodo Architecture

- GitOps repo `/workspace/homelab-stacks` (`klinker41/homelab-stacks`) drives
  all 34 Komodo v2 stacks across 6 environments: `unraid-nas`,
  `unraid-seedbox`, `unraid-aibox` (ID `22`), `pi-studio`, `pi-lab`, and
  `pi-swarm`.
- Compose files reside in `stacks/<host>/<stack>/compose.yaml` using
  `${VAR_NAME}` substitution; `syncs/homelab.toml` maps stack `environment`
  entries to encrypted/masked Komodo `Variable` resources via
  `VAR_NAME=[[KOMODO_SECRET_NAME]]`.
- Pushing to `main` triggers the two-stage `gitops-deploy` Komodo Procedure at
  https://komodo.klinker-cabin.computer/listener/github/procedure/gitops-deploy
  — Stage 1 runs `RunSync` (`homelab-sync`) and Stage 2 runs
  `BatchDeployStackIfChanged` (`Deploy Changed Stacks`,
  `pattern = "nas-*,seedbox-*,pi-studio-*"`, omitting `aibox-*` since AIBox is
  powered off when idle). Because `gitops-deploy` locks itself while running,
  edits to `gitops-deploy` in `syncs/homelab.toml` require running
  `km execute -y run-sync homelab-sync` directly on `komodo-core`.
- Komodo Core (`komodo-core`, `komodo-mongo`, `komodo-periphery`) runs on
  `unraid-nas` (`stacks/unraid-nas/komodo/compose.yaml`), polling server
  metrics every 15s (`KOMODO_MONITORING_INTERVAL: 15-sec`) and managing remote
  servers and Swarms via Resource Sync. On `unraid-nas`, Periphery clones
  stacks under
  `/mnt/user/docker/komodo/periphery/stacks/<stack>/stacks/unraid-nas/<subpath>`
  (e.g., `.../nas-management/stacks/unraid-nas/management`). After a power
  outage or `nas` reboot, remote `komodo-periphery` agents (e.g.,
  `unraid-seedbox`) may transiently report `Server <id> is not connected`
  while reconnecting even when the host and array are healthy.
- Scheduled `Global Auto Update` procedure (`syncs/homelab.toml`) runs daily
  at 03:00 local time, stopping and recreating stacks configured with
  `poll_for_updates = true` and `auto_update = true` (e.g., `pi-studio-files`).

## Site-to-Site WireGuard VPN (`gluetun`)

- Cross-site connectivity to Boulder (`klinker.computer` UDP `51820`, updated
  via `cloudflare-ddns`) uses 4 Gluetun WireGuard services (`gluetun-home` and
  `gluetun-home-backup` across `unraid-nas`, `unraid-seedbox`, and
  `unraid-aibox`).
- Because Gluetun WireGuard configs require a raw IPv4 `Endpoint` rather than
  a hostname, all 4 services mount their host config to `/tmp/wg0.conf:ro` and
  run an inline `/bin/sh` entrypoint in `compose.yaml` that resolves
  `klinker.computer` into `/gluetun/wireguard/wg0.conf` at startup and every 5
  minutes, restarting `/gluetun-entrypoint` in-place without dropping the
  container network namespace.
- Containers sharing a Gluetun namespace via `network_mode`
  (`service:<gluetun-service>` or `container:<gluetun-container>`) must be
  recreated via Docker Compose whenever their upstream Gluetun container gets
  a new container ID. Dependents include `semaphore-boulder` in
  `nas-management` on `unraid-nas`
  (`https://ansible-boulder.klinker-cabin.computer`, Uptime Kuma monitor ID
  `68`, port `3008:3000` via `service:gluetun-home`) and `rsync-backup`
  (`nas-files` on `unraid-nas` and `seedbox-files` on `unraid-seedbox` via
  `service:gluetun-home-backup`).

## Raspberry Pi Clusters & Ansible Management

- Ansible playbooks for `Pi Swarm` and `Pi Lab` live on `unraid-nas` under
  `/mnt/user/docker/ansible-estes/main/` and
  `/mnt/user/docker/ansible-boulder/main/` (`install-docker.yml`,
  `remove-docker.yml`, `setup-docker.yml`, `configure-reboot-scripts.yml`,
  `setup-nas.yml`). `setup-nas.yml` checks I2C device `0x5a` and configures
  the ATX Power Management HAT with watchdog enabled
  (`i2cset -y 1 0x5a 0x35 0x80`) and always-on auto power restore
  (`i2cset -y 1 0x5a 0x34 0x03`).
- Both `Pi Swarm` (`pi-rack-1`, `PERIPHERY_CONNECT_AS=pi-swarm-node`) and
  `Pi Lab` (`pi-lab-nas`, `PERIPHERY_CONNECT_AS=pi-lab-node`) run
  `komodo_periphery` as a Swarm service constrained to `node.role == manager`
  with keys persisted at `/etc/komodo/keys` (`portainer_agent` removed).
- **Pi Swarm (19 nodes)**: `pi-rack-1` (`192.168.1.78`) is Swarm manager;
  `pi-rack-5` (`primary-storage=true, storage=true`) and `pi-rack-6..8`
  (`storage=true`) are storage workers; `pi-rack-2..4`, `pi-rack-9..10`,
  `pi-rack-cm-1..3`, and 6 `super6c-1-*` nodes are compute workers.
- **Pi Lab**: `pi-lab-control-1` (`192.168.1.92`) reaches the LAN via bridge
  `br0` (`eth0` and `eth1`) on `pi-lab-nas` (`192.168.1.113`). `pi-lab-nas`
  uses `/etc/systemd/system/br0-forward-allow.service`
  (`PartOf=docker.service`, `WantedBy=multi-user.target docker.service`) to
  insert `iptables -I FORWARD -i br0 -j ACCEPT` and `-o br0 -j ACCEPT` rules
  so Docker's default `FORWARD DROP` policy does not block bridged traffic.

## Host-Specific Stack Notes

- **`unraid-nas` (`192.168.1.95`)**:
  - `jenkins-docker-agent` (`stacks/unraid-nas/buildbox/compose.yaml`) mounts
    the host Docker socket, spawning intermediate `docker build` containers
    directly on the NAS host.
  - `nas-reverse-proxy` (`stacks/unraid-nas/reverse-proxy/compose.yaml`) runs
    `nginx-proxy-manager` (admin API port `81`, SQLite DB at
    `/mnt/user/docker/nginx-proxy-manager/data/database.sqlite`),
    `cloudflare-ddns`, `crowdsec`, `crowdsec-web-ui`, and `cs-unifi-bouncer`
    (syncing CrowdSec firewall groups to UniFi Network at
    `https://192.168.1.181:443`). On `crowdsec` restart, LAPI (port `8080`)
    takes ~9s to initialize, causing `cs-unifi-bouncer`
    (`http://crowdsec:8080/v1/decisions/stream`) to briefly restart-loop and
    trigger a transient Komodo unhealthy alert.
  - `nas-ai` (`stacks/unraid-nas/ai/compose.yaml`) mounts
    `/mnt/user/media/Reels:/reels` (downloaded Instagram Reels) and
    `/mnt/user/docker/Instagram:/instagram` (`cookies.txt`) in Antigravity.
- **`unraid-seedbox` (`192.168.1.46`)**:
  - `seedbox-torrent` (`stacks/unraid-seedbox/torrent/compose.yaml`) runs
    `qbittorrent` (`binhex/arch-qbittorrentvpn` with PIA: `VPN_PROV=pia`,
    `VPN_CLIENT=wireguard`, `STRICT_PORT_FORWARD=yes`), `qbittorrent-private`,
    `qui` (healthcheck `localhost:7476`), `mousetrap` (healthcheck
    `127.0.0.1:39842`), and `upload-assistant`.
- **`unraid-aibox`**:
  - Runs `aibox-ai` (including `Strata`) and `aibox-gaming`. In `aibox-gaming`,
    `wolf` and `wolf-den` share `/var/run/wolf/wolf.sock`; `wolf-den` requires
    `/var/run/wolf:rw` and write permissions on `wolf.sock` for its ASP.NET
    Core process to connect to the Wolf API.
  - Syncing `unraid-aibox` backups via `rsync-backup` to
    `192.168.1.72:/mnt/external_hdd/backup/aibox/` as `jklinker` requires
    `--omit-dir-times` (`-O`) if the destination top-level directory is owned
    by `root:root` to avoid non-zero exit codes from timestamp warnings.
- **`pi-studio` (`192.168.1.72`, Boulder)**:
  - Runs on a 13.8 GiB SD card rootfs with `/mnt/external_hdd` and no
    site-to-site WireGuard route to Estes LAN. Configured in Komodo with
    inbound WebSocket (`address: ""`), where `komodo-periphery` connects
    outbound to `komodo-core` on `unraid-nas` (port `9120`,
    `wss://komodo.klinker-cabin.computer/ws/periphery?server=pi-studio`),
    polled hourly (`KOMODO_RESOURCE_POLL_INTERVAL: 1-hr`).
  - Hosts `pi-studio-dashboard` (`uptime-kuma` at
    `https://uptime.klinker.computer` with SQLite DB
    `/home/jklinker/uptime/kuma.db`, and `glances`), `pi-studio-reverse-proxy`
    (`nginx-proxy-manager` / `openresty`, `cloudflare-ddns`, `crowdsec`,
    `cs-unifi-bouncer`, `crowdsec-web-ui`), and `pi-studio-files`
    (`filebrowser-quantum` at `https://files.klinker.computer` and
    `rsync-backup`).
  - Uptime Kuma's `Estes Battery` monitor (ID `80`, `Server` group, `all`
    status page) polls `https://home-api.klinker-cabin.computer/api/solar`
    every 120s (48s timeout, 2 retries) via `HTTP(s) - Json Query` to alert
    Slack when the Estes battery drops below 50%.
  - `filebrowser-quantum`
    (`/home/jklinker/filebrowser-quantum/data/config.yaml`) configures
    `/srv/home` and `/srv/backup` (from `/mnt/external_hdd`) with
    `rules: [{ folderPath: "/", viewable: true }]` so directories browse
    on-demand (`GetFsInfoViewableOnly`) without creating a container-layer
    SQLite index (`/home/filebrowser/tmp/sql/index_all.db`) or waking the
    spun-down `/mnt/external_hdd` drive.

- On `unraid-nas` (`stacks/unraid-nas/files/compose.yaml`), the `rsync-backup`
container runs daily at 1:00 AM (`America/Denver`, `RSYNC_CRONTAB=0 1 * * *`)
over `gluetun-home-backup` to `192.168.1.72:/mnt/external_hdd/backup/server`
using `-rtv --timeout=7200 --delete`.
- Unraid `rsync-backup` paths mapped under `/mnt/external_hdd/backup/server`:
`/mnt/user/ai` -> `ai`, `/mnt/user/docker` -> `docker`, `/mnt/user/downloads` ->
`downloads`, `/mnt/user/gaming` -> `gaming`, `/mnt/user/home` -> `homes`,
`/mnt/user/photos` -> `photos`, and `/boot` (Unraid flash drive) -> `unraid`.

- On `unraid-nas`, both `jellyfin`
(`stacks/unraid-nas/media-playback/compose.yaml`) and `antigravity`
(`stacks/unraid-nas/ai/compose.yaml`) run without Docker memory limits
(planned 96 GiB RAM upgrade).
- Jellyfin on unraid-nas sets
<ParallelImageEncodingLimit>2</ParallelImageEncodingLimit> in
/mnt/user/docker/jellyfin/config/config/system.xml to prevent OOM crashes when
generating thumbnails for ultra-high-resolution photos in
/mnt/user/home/jklinker/Photo Shoots.
- Pushing changes to stacks/unraid-nas/ai/compose.yaml triggers Komodo GitOps to
recreate the antigravity container, disconnecting active agent sessions.

- Configured GitHub App credentials on the `antigravity` container in
`stacks/unraid-nas/ai/compose.yaml` via `GITHUB_APP_ID=5864331`,
`GITHUB_APP_INSTALLATION_ID=184407633`, and
`GITHUB_APP_PRIVATE_KEY_PATH=/home/developer/.gemini/jklinker-antigravity-bot.2026-11-21.private-key.pem`.

- unraid-aibox (192.168.1.57, NIC MACs 1a:4b:24:c7:26:c5 and bc:fc:e7:3b:d2:f0)
stays powered off when idle, wakes via Wake-on-LAN from unraid-nas
(192.168.1.95), and requires re-arming WoL on eth0 and eth1 after every boot or
reboot.
- unraid-aibox hosts five Komodo-managed Docker stacks (aibox-dashboard,
aibox-files, aibox-gaming, aibox-photos, aibox-ai) on a 50GB docker.img;
recreating gluetun-home-backup during deployment requires restarting
rsync-backup afterward.

- The jellyfin (stacks/unraid-nas/media-playback/compose.yaml, Komodo stack
nas-media-playback) and antigravity (stacks/unraid-nas/ai/compose.yaml, Komodo
stack nas-ai) services run without Docker memory limits; tests/test_stacks.py
enforces the absence of deploy, memory, mem_limit, and memswap_limit keys on
both services.

- The strata service in stacks/unraid-aibox/ai/compose.yaml (deployed via Komodo
as stack aibox-ai) is configured for 32 GB system RAM using the
Qwen3.8-Flash-Next Coder model (FAMILY=coder, MODEL=IQ1_M, CONTEXT=131072,
KV=k8v4, VISION=no, REINSTALL=1).
- Strata model weights and packs on unraid-aibox are persisted on host storage
under /mnt/user/ai/strata/models/ and /mnt/user/ai/strata/packs/.

- The `buildbox` stack (`stacks/unraid-nas/buildbox/compose.yaml`) on
`unraid-nas` (`192.168.1.95`) runs `jenkins` alongside `jenkins-android-agent`
and `jenkins-docker-agent`, mounting `/mnt/user/docker/jenkins/home` to
`/var/jenkins_home` to persist controller state and archived build artifacts
(`jobs/<job>/builds/<build-number>/archive`) copied back from ephemeral agent
volumes.

- Jenkins (https://jenkins.klinker.computer) runs in Docker on unraid-nas
(192.168.1.95, also referred to as unraid-server) with job configs at
/mnt/user/docker/jenkins/home/jobs/<job>/config.xml (loaded cleanly by
restarting the jenkins container via Komodo), GitHub credentials ID klinker41,
and Android build agent unraid-server-android-agent (JDK 21, android-37.2,
build-tools/37.0.0).

- Dedicated Watchtower stacks (`nas-watchtower`, `seedbox-watchtower`,
`aibox-watchtower`, `pi-studio-watchtower`) run `nickfedor/watchtower:latest`
(`WATCHTOWER_SCHEDULE=0 0 4 * * *`, `WATCHTOWER_CLEANUP=true`) across
`unraid-nas`, `unraid-seedbox`, `unraid-aibox` (`send_alerts = false`), and
`pi-studio`, scoped strictly to auto-update `komodo-core` and
`komodo-periphery`.
- `komodo-core` and `komodo-periphery` are intentionally excluded from Komodo's
`GlobalAutoUpdate` (via `komodo.skip` labels and `PROTECTED_CONTAINERS`) so
`komodo-periphery` cannot kill itself mid-deployment, with
`tests/test_stacks.py` enforcing that Watchtower targets only Komodo services.

- Jenkins runs as the jenkins container on unraid-nas (192.168.1.95) with its
persistent home directory at /mnt/user/docker/jenkins/home (UID/GID 1000:1000)
and build agents unraid-server-android-agent and unraid-server-docker-agent.

- autobrr runs on unraid-nas (192.168.1.95) at
https://autobrr.klinker-cabin.computer with its SQLite database stored at
/mnt/user/docker/autobrr/config/autobrr.db.
- mousetrap runs on unraid-seedbox and uses an autobrr API key to update the MAM
session ID in autobrr, while Homepage uses a dedicated read-scoped API key to
poll autobrr stats.

- Komodo (`https://komodo.klinker-cabin.computer`) routes Slack alerts through
the `slack-homelab` alerter with an explicit problem-only `alert_types`
allowlist so routine `Global Auto Update` (03:00), `ScheduleRun`
(`Backup Core Database`, `Rotate Server Keys`), and `homelab-sync` update events
are suppressed.

---
topic: homelab-stacks
category: projects
tags: [komodo, gitops, docker, stacks, homelab]
updated_at: 2026-11-28T07:41:20.948191+00:00
compacted_at: 2026-11-28T07:41:20.948191+00:00
confidence: 1.0
---

# Homelab-Stacks

## GitOps & Komodo Architecture

- GitOps repo `/workspace/homelab-stacks` (`klinker41/homelab-stacks`) drives
  all 34 Komodo v2 stacks across 6 environments: `unraid-nas`,
  `unraid-seedbox`, `unraid-aibox` (ID `22`), `pi-studio`, `pi-lab`, and
  `pi-swarm`.
- Compose files reside at `stacks/<host>/<stack>/compose.yaml` using
  `${VAR_NAME}` substitution; `syncs/homelab.toml` maps stack `environment`
  entries to encrypted/masked Komodo `Variable` resources via
  `VAR_NAME=[[KOMODO_SECRET_NAME]]`.
- `homelab-sync` Resource Sync (`syncs/homelab.toml`) uses
  `match_tags = ["gitops"]`; live UI/API edits to `gitops`-tagged resources
  (such as `unraid-nas`) must be committed to `syncs/homelab.toml` to avoid
  being reverted.
- Pushing to `main` triggers the two-stage `gitops-deploy` Komodo Procedure at
  https://komodo.klinker-cabin.computer/listener/github/procedure/gitops-deploy
  — Stage 1 runs `RunSync` (`homelab-sync`) and Stage 2 runs
  `BatchDeployStackIfChanged` (`Deploy Changed Stacks`,
  `pattern = "nas-*,seedbox-*,pi-studio-*"`, omitting `aibox-*` since AIBox is
  powered off when idle). Because `gitops-deploy` locks itself while running,
  edits to `gitops-deploy` in `syncs/homelab.toml` require running
  `km execute -y run-sync homelab-sync` directly on `komodo-core`.
- Komodo Core (`komodo-core`, `komodo-mongo`, `komodo-periphery`) runs on
  `unraid-nas` (`stacks/unraid-nas/komodo/compose.yaml`, host
  `/mnt/user/docker/komodo/compose.yaml`), polling server metrics every 15s
  (`KOMODO_MONITORING_INTERVAL: 15-sec`) and managing remote servers and Swarms
  via Resource Sync. On `unraid-nas`, Periphery clones stacks under
  `/mnt/user/docker/komodo/periphery/stacks/<stack>/stacks/unraid-nas/<subpath>`
  (e.g., `.../nas-management/stacks/unraid-nas/management`). After a power
  outage or `nas` reboot, remote `komodo-periphery` agents (e.g.,
  `unraid-seedbox`) may transiently report `Server <id> is not connected` while
  reconnecting even when the host and array are healthy.
- **Auto-Updates & Watchtower**: Scheduled `Global Auto Update`
  (`syncs/homelab.toml`) runs daily at 03:00 local time, stopping and
  recreating stacks with `poll_for_updates = true` and `auto_update = true`
  (e.g., `pi-studio-files`). `komodo-core` and `komodo-periphery` are excluded
  from `GlobalAutoUpdate` (via `komodo.skip` labels and `PROTECTED_CONTAINERS`)
  so Periphery cannot kill itself mid-deploy; dedicated Watchtower stacks
  (`nas-watchtower`, `seedbox-watchtower`, `aibox-watchtower`,
  `pi-studio-watchtower`) run `nickfedor/watchtower:latest`
  (`WATCHTOWER_SCHEDULE=0 0 4 * * *`, `WATCHTOWER_CLEANUP=true`; `unraid-aibox`
  sets `send_alerts = false`) strictly scoped to `komodo-core` and
  `komodo-periphery`, enforced by `tests/test_stacks.py`.
- **Alerting**: Komodo (`https://komodo.klinker-cabin.computer`) routes Slack
  alerts via `slack-homelab` (`cpu_warning: 90%`, `cpu_critical: 99%` on
  `unraid-nas`, `unraid-seedbox`, and `pi-studio`; `unraid-nas` sets
  `send_cpu_alerts = false` under `[server.config]` in `syncs/homelab.toml`).
  `slack-homelab` uses an explicit problem-only `alert_types` allowlist so
  routine `Global Auto Update` (03:00), `ScheduleRun` (`Backup Core Database`,
  `Rotate Server Keys`), and `homelab-sync` update events are suppressed; it
  has no `gitops` tag or `[[alerter]]` entry in `syncs/homelab.toml`, so its
  failure-only config persists directly in Komodo.

## Site-to-Site WireGuard VPN (`gluetun`)

- Cross-site connectivity to Boulder (`klinker.computer` UDP `51820`, updated
  via `cloudflare-ddns`) uses 4 Gluetun WireGuard services (`gluetun-home` and
  `gluetun-home-backup` across `unraid-nas`, `unraid-seedbox`, and
  `unraid-aibox`).
- Because Gluetun WireGuard configs require a raw IPv4 `Endpoint` rather than a
  hostname, all 4 services mount their host config to `/tmp/wg0.conf:ro` and
  run an inline `/bin/sh` entrypoint in `compose.yaml` that resolves
  `klinker.computer` into `/gluetun/wireguard/wg0.conf` at startup and every 5
  minutes, restarting `/gluetun-entrypoint` in-place without dropping the
  container network namespace.
- Containers sharing a Gluetun namespace via `network_mode`
  (`service:<gluetun-service>` or `container:<gluetun-container>`) must be
  recreated via Docker Compose whenever their upstream Gluetun container gets a
  new container ID. Dependents include `semaphore-boulder` in `nas-management`
  on `unraid-nas` (`https://ansible-boulder.klinker-cabin.computer`, Uptime
  Kuma monitor ID `68`, port `3008:3000` via `service:gluetun-home`) and
  `rsync-backup` (`nas-files` on `unraid-nas`, `seedbox-files` on
  `unraid-seedbox`, and `aibox-files` on `unraid-aibox` via
  `service:gluetun-home-backup`).

## Raspberry Pi Clusters & Ansible Management

- Ansible playbooks for `Pi Swarm` and `Pi Lab` live on `unraid-nas` under
  `/mnt/user/docker/ansible-estes/main/` and
  `/mnt/user/docker/ansible-boulder/main/` (`install-docker.yml`,
  `remove-docker.yml`, `setup-docker.yml`, `configure-reboot-scripts.yml`,
  `setup-nas.yml`). `setup-nas.yml` checks I2C device `0x5a` and configures the
  ATX Power Management HAT with watchdog enabled (`i2cset -y 1 0x5a 0x35 0x80`)
  and always-on auto power restore (`i2cset -y 1 0x5a 0x34 0x03`).
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
  insert `iptables -I FORWARD -i br0 -j ACCEPT` and `-o br0 -j ACCEPT` rules so
  Docker's default `FORWARD DROP` policy does not block bridged traffic.

## Host-Specific Stack Notes

### `unraid-nas` (`192.168.1.95`, also `unraid-server`)

- **Reverse Proxy & CrowdSec** (`nas-reverse-proxy`,
  `stacks/unraid-nas/reverse-proxy/compose.yaml`): Runs `nginx-proxy-manager`
  (admin API port `81`, SQLite DB at
  `/mnt/user/docker/nginx-proxy-manager/data/database.sqlite`),
  `cloudflare-ddns`, `crowdsec` (config `/mnt/user/docker/crowdsec/config`),
  `crowdsec-web-ui`, and `cs-unifi-bouncer`. `crowdsec` + `cs-unifi-bouncer`
  sync active bans into a Cabin UniFi firewall drop/block group at
  `https://192.168.1.181:443`, protecting WAN ingress across all
  `.klinker-cabin.computer` services (including `unraid-seedbox` containers).
  On `crowdsec` restart, LAPI (port `8080`) takes ~9s to initialize, causing
  `cs-unifi-bouncer` (`http://crowdsec:8080/v1/decisions/stream`) to briefly
  restart-loop and trigger a transient Komodo unhealthy alert. Local
  `s02-enrich` parser `homelab/immich-whitelist`
  (`/mnt/user/docker/crowdsec/config/parsers/s02-enrich/immich-whitelist.yaml`)
  whitelists `GET` 404 responses on `photos.klinker-cabin.computer` matching
  `(?i)^/api/assets/[a-f0-9-]+/thumbnail([?]|$)`.
- **AI / Antigravity** (`nas-ai`, `stacks/unraid-nas/ai/compose.yaml`): Runs
  `antigravity` without Docker memory limits (planned 96 GiB RAM upgrade;
  `tests/test_stacks.py` enforces absence of `deploy`, `memory`, `mem_limit`,
  and `memswap_limit` on both `antigravity` and `jellyfin`). Stays in `nas-ai`
  rather than `nas-buildbox` because `antigravity-plugin` reaches
  `http://homelab-ops:5005/sse` via `ai_default`, `ANTIGRAVITY_*` secrets are
  scoped to `nas-ai` in `syncs/homelab.toml`, and separation avoids restarting
  `antigravity` on `nas-buildbox` deploys; pushing edits to
  `stacks/unraid-nas/ai/compose.yaml` triggers GitOps to recreate
  `antigravity`, disconnecting active sessions. Mounts
  `/mnt/user/media/Reels:/reels` (downloaded Instagram Reels),
  `/mnt/user/docker/Instagram:/instagram` (`cookies.txt`),
  `/mnt/user/build/antigravity/{gradle,m2}` to `/home/developer/{.gradle,.m2}`
  (`GRADLE_USER_HOME=/home/developer/.gradle`), and
  `/mnt/user/docker/gradle-cache/init.d` read-write to
  `/home/developer/.gradle/init.d` (without `:ro` so `entrypoint.sh`'s
  `chown -R` succeeds), joining the `gradle-cache` network (`external: true`).
  GitHub App credentials: `GITHUB_APP_ID=5864331`,
  `GITHUB_APP_INSTALLATION_ID=184407633`, and `GITHUB_APP_PRIVATE_KEY_PATH`:
  `/home/developer/.gemini/jklinker-antigravity-bot.2026-11-21.private-key.pem`.
- **Buildbox, Jenkins & Gradle Cache** (`nas-buildbox`,
  `stacks/unraid-nas/buildbox/compose.yaml`): The cache-backed
  `/mnt/user/build` share keeps heavy build caches and workspaces off
  `docker.img` and outside nightly `/mnt/user/docker` `rsync-backup` runs
  (`filebrowser-quantum` mounts `/mnt/user/build` at `/srv/build`). Runs
  `jenkins` (`https://jenkins.klinker.computer`), `jenkins-android-agent`
  (`unraid-server-android-agent`: JDK 21, `android-37.2`,
  `build-tools/37.0.0`), `jenkins-docker-agent` (`unraid-server-docker-agent`,
  mounts host Docker socket to spawn intermediate `docker build` containers
  directly on NAS), and `gradle-cache` (`gradle/build-cache-node:latest`).
  `jenkins` mounts `/mnt/user/docker/jenkins/home` (UID/GID `1000:1000`) to
  `/var/jenkins_home` for controller state, job configs
  (`jobs/<job>/config.xml`, reloaded cleanly by restarting `jenkins` via
  Komodo), GitHub credentials ID `klinker41`, and archived artifacts
  (`jobs/<job>/builds/<build-number>/archive`) copied back from ephemeral agent
  volumes. `jenkins-android-agent` bind-mounts
  `/mnt/user/build/jenkins/{gradle,work,m2}` (`1000:1000`) to
  `/home/jenkins/{.gradle,work,.m2}` and
  `/mnt/user/docker/gradle-cache/init.d:/home/jenkins/.gradle/init.d:ro`.
  `gradle-cache` mounts `/mnt/user/build/gradle-cache-node:/data` and
  `/mnt/user/docker/gradle-cache/conf:/data/conf` (no host ports published),
  defines the `gradle-cache` network, and shares
  `/mnt/user/docker/gradle-cache/init.d/remote-cache.gradle.kts` targeting
  `http://gradle-cache:5071/cache/`.
- **Media & Automation**:
  - `nas-media-playback` (`stacks/unraid-nas/media-playback/compose.yaml`):
    `jellyfin` runs without Docker memory limits (enforced alongside
    `antigravity` in `tests/test_stacks.py`) and sets
    `<ParallelImageEncodingLimit>2</ParallelImageEncodingLimit>` in
    `/mnt/user/docker/jellyfin/config/config/system.xml` to prevent OOM crashes
    when generating thumbnails for ultra-high-resolution photos in
    `/mnt/user/home/jklinker/Photo Shoots`.
  - `nas-media-pipeline-processing` (Compose project
    `media-pipeline-processing`): runs `tdarr-server` (Compose service
    `tdarr`), `tdarr-node` (registered as `unraid-server` with QSV/VAAPI
    encoding), `metadata-remote`, and `rename`.
  - `autobrr` (`https://autobrr.klinker-cabin.computer`, SQLite DB
    `/mnt/user/docker/autobrr/config/autobrr.db`): `mousetrap` on
    `unraid-seedbox` updates the MAM session ID via an autobrr API key, while
    Homepage polls stats using a dedicated read-scoped API key.
- **Files & Backups** (`nas-files`, `stacks/unraid-nas/files/compose.yaml`):
  `rsync-backup` runs daily at 01:00 `America/Denver`
  (`RSYNC_CRONTAB=0 1 * * *`) over `gluetun-home-backup` to
  `192.168.1.72:/mnt/external_hdd/backup/server` using
  `-rtv --timeout=7200 --delete`. Mappings under
  `/mnt/external_hdd/backup/server`: `/mnt/user/ai` -> `ai`,
  `/mnt/user/docker` -> `docker`, `/mnt/user/downloads` -> `downloads`,
  `/mnt/user/gaming` -> `gaming`, `/mnt/user/home` -> `homes`,
  `/mnt/user/photos` -> `photos`, and `/boot` (Unraid flash drive) -> `unraid`.

### `unraid-seedbox` (`192.168.1.46`)

- **`seedbox-torrent`** (`stacks/unraid-seedbox/torrent/compose.yaml`): Runs
  `qbittorrent` (`binhex/arch-qbittorrentvpn`, WebUI port `8081` behind
  `https://qbittorrent.klinker-cabin.computer`,
  `/mnt/user/docker/qbittorrent/config/wireguard/wg0.conf`, PIA:
  `VPN_PROV=pia`, `VPN_CLIENT=wireguard`, `STRICT_PORT_FORWARD=yes`, endpoint
  `netherlands.pvt.site:1337`), `qbittorrent-private`, `qui`
  (`https://qui.klinker-cabin.computer`, healthcheck `localhost:7476`),
  `mousetrap` (healthcheck `127.0.0.1:39842`), and `upload-assistant`.

### `unraid-aibox` (`192.168.1.57`)

- Stays powered off when idle, wakes via Wake-on-LAN from `unraid-nas`
  (`192.168.1.95`; NIC MACs `1a:4b:24:c7:26:c5` and `bc:fc:e7:3b:d2:f0`), and
  requires re-arming WoL on `eth0` and `eth1` after every boot or reboot.
- Hosts 5 Komodo stacks (`aibox-dashboard`, `aibox-files`, `aibox-gaming`,
  `aibox-photos`, `aibox-ai`, plus `aibox-watchtower`) on a 50 GB `docker.img`.
- **`aibox-ai`** (`stacks/unraid-aibox/ai/compose.yaml`): Runs `strata`
  configured for 32 GB system RAM with Qwen3.8-Flash-Next Coder
  (`FAMILY=coder`, `MODEL=IQ1_M`, `CONTEXT=131072`, `KV=k8v4`, `VISION=no`,
  `REINSTALL=1`); model weights and packs persist under
  `/mnt/user/ai/strata/models/` and `/mnt/user/ai/strata/packs/`.
- **`aibox-gaming`**: `wolf` and `wolf-den` share `/var/run/wolf/wolf.sock`;
  `wolf-den` requires `/var/run/wolf:rw` and write permissions on `wolf.sock`
  for its ASP.NET Core process to connect to the Wolf API.
- **`aibox-files`**: Syncing backups via `rsync-backup` to
  `192.168.1.72:/mnt/external_hdd/backup/aibox/` as `jklinker` requires
  `--omit-dir-times` (`-O`) if the destination top-level directory is owned by
  `root:root` to avoid non-zero exit codes from timestamp warnings.

### `pi-studio` (`192.168.1.72`, Boulder)

- Runs on a 13.8 GiB SD card rootfs with `/mnt/external_hdd` and no
  site-to-site WireGuard route to Estes LAN. Configured in Komodo with inbound
  WebSocket (`address: ""`), where `komodo-periphery` connects outbound to
  `komodo-core` on `unraid-nas` (port `9120`,
  `wss://komodo.klinker-cabin.computer/ws/periphery?server=pi-studio`), polled
  hourly (`KOMODO_RESOURCE_POLL_INTERVAL: 1-hr`).
- Hosts `pi-studio-dashboard` (`uptime-kuma` at
  `https://uptime.klinker.computer`, SQLite DB `/home/jklinker/uptime/kuma.db`,
  and `glances`), `pi-studio-reverse-proxy` (`nginx-proxy-manager` /
  `openresty`, `cloudflare-ddns`, `crowdsec`, `cs-unifi-bouncer`,
  `crowdsec-web-ui`), and `pi-studio-files` (`filebrowser-quantum` at
  `https://files.klinker.computer` and `rsync-backup`).
- Uptime Kuma's `Estes Battery` monitor (ID `80`, `Server` group, `all` status
  page) polls `https://home-api.klinker-cabin.computer/api/solar` every 120s
  (48s timeout, 2 retries) via `HTTP(s) - Json Query` to alert Slack when the
  Estes battery drops below 50%.
- `filebrowser-quantum` (`/home/jklinker/filebrowser-quantum/data/config.yaml`)
  configures `/srv/home` and `/srv/backup` (from `/mnt/external_hdd`) with
  `rules: [{ folderPath: "/", viewable: true }]` so directories browse
  on-demand (`GetFsInfoViewableOnly`) without creating a container-layer SQLite
  index (`/home/filebrowser/tmp/sql/index_all.db`) or waking the spun-down
  `/mnt/external_hdd` drive.

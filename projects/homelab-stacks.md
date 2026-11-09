---
topic: homelab-stacks
category: projects
tags: [komodo, gitops, docker, stacks, homelab]
updated_at: 2026-11-09T07:31:12.522129+00:00
compacted_at: 2026-11-09T07:31:12.522129+00:00
confidence: 1.0
---

# Homelab-Stacks

## GitOps & Komodo Architecture

- GitOps repository at `/workspace/homelab-stacks` (`klinker41/homelab-stacks`)
  drives all 34 Komodo v2 stacks across 6 environments: `unraid-nas`,
  `unraid-seedbox`, `unraid-aibox` (ID `22`), `pi-studio`, `pi-lab`, and
  `pi-swarm`.
- Compose files reside in `stacks/<host>/<stack>/compose.yaml` using
  `${VAR_NAME}` substitution; `syncs/homelab.toml` maps stack `environment`
  entries to encrypted/masked Komodo `Variable` resources via
  `VAR_NAME=[[KOMODO_SECRET_NAME]]`.
- Pushing to `main` triggers the `gitops-deploy` Komodo Procedure
  (`https://komodo.klinker-cabin.computer/listener/github/procedure/gitops-deploy`),
  executing `RunSync` (`homelab-sync`) followed by `BatchDeployStackIfChanged`
  (`pattern = "*"`).
- Komodo Core (`komodo-core`, `komodo-mongo`, `komodo-periphery`) runs on
  `unraid-nas` (`stacks/unraid-nas/komodo/compose.yaml`), polling server
  metrics every 15 seconds (`KOMODO_MONITORING_INTERVAL: 15-sec`) and managing
  all remote servers and Docker Swarms via declarative Resource Sync.
- On `unraid-nas`, Komodo Periphery clones stack directories to
  `/mnt/user/docker/komodo/periphery/stacks/<stack-name>/stacks/unraid-nas/<subpath>`
  (e.g., `.../nas-management/stacks/unraid-nas/management`).
- A scheduled `Global Auto Update` Komodo procedure (`syncs/homelab.toml`) runs
  daily at 03:00 local time, briefly stopping and recreating stacks configured
  with `poll_for_updates = true` and `auto_update = true` (e.g.,
  `pi-studio-files`).

## Site-to-Site WireGuard VPN (`gluetun`)

- Cross-site connectivity to the Boulder network (`klinker.computer` on UDP
  port `51820`, updated dynamically via `cloudflare-ddns`) is provided by 4
  Gluetun WireGuard services (`gluetun-home` and `gluetun-home-backup` across
  `unraid-nas`, `unraid-seedbox`, and `unraid-aibox`).
- Because Gluetun WireGuard configs require a raw IPv4 `Endpoint` rather than a
  hostname, all 4 services mount their host config to `/tmp/wg0.conf:ro` and
  use an inline `/bin/sh` entrypoint in `compose.yaml` to resolve
  `klinker.computer` into `/gluetun/wireguard/wg0.conf` at startup and every 5
  minutes, restarting `/gluetun-entrypoint` in-place without dropping the
  container network namespace.
- Dependent containers sharing a Gluetun network namespace via `network_mode`
  (`service:<gluetun-service>` or `container:<gluetun-container>`)—such as
  `semaphore-boulder` (Ansible Boulder on port `:3000` in `nas-management` on
  `unraid-nas`, serving `https://ansible-boulder.klinker-cabin.computer` via
  `service:gluetun-home`) and `rsync-backup` (`nas-files` on `unraid-nas` and
  `seedbox-files` on `unraid-seedbox` via `service:gluetun-home-backup`)—must
  be restarted or recreated whenever their parent Gluetun container is updated
  or recreated with a new container ID so they rebind to the new network
  namespace.

## Raspberry Pi Clusters & Ansible Management

- Ansible playbooks managing `Pi Swarm` and `Pi Lab` are stored on `unraid-nas`
  under `/mnt/user/docker/ansible-estes/main/` and
  `/mnt/user/docker/ansible-boulder/main/` (`install-docker.yml`,
  `remove-docker.yml`, `setup-docker.yml`, `configure-reboot-scripts.yml`, and
  `setup-nas.yml`).
- `setup-nas.yml` checks for I2C device `0x5a` and configures the ATX Power
  Management HAT with watchdog enabled (`i2cset -y 1 0x5a 0x35 0x80`) and
  always-on auto power restore (`i2cset -y 1 0x5a 0x34 0x03`).
- Both `Pi Swarm` (`pi-rack-1`, `PERIPHERY_CONNECT_AS=pi-swarm-node`) and
  `Pi Lab` (`pi-lab-nas`, `PERIPHERY_CONNECT_AS=pi-lab-node`) run
  `komodo_periphery` as a Docker Swarm service constrained to
  `node.role == manager` with keys persisted at `/etc/komodo/keys`
  (`portainer_agent` is completely removed).
- **Pi Swarm (19 nodes)**: `pi-rack-1` (`192.168.1.78`) is the Swarm manager;
  `pi-rack-5` (`primary-storage=true, storage=true`) and `pi-rack-6..8`
  (`storage=true`) are storage workers; `pi-rack-2..4`, `pi-rack-9..10`,
  `pi-rack-cm-1..3`, and 6 `super6c-1-*` nodes are compute workers.
- **Pi Lab**: `pi-lab-control-1` (`192.168.1.92`) reaches the LAN through a
  network bridge (`br0` spanning `eth0` and `eth1`) on `pi-lab-nas`
  (`192.168.1.113`). `pi-lab-nas` uses
  `/etc/systemd/system/br0-forward-allow.service` (`PartOf=docker.service`,
  `WantedBy=multi-user.target docker.service`) to insert
  `iptables -I FORWARD -i br0 -j ACCEPT` and `-o br0 -j ACCEPT` rules so
  Docker's default `FORWARD DROP` policy does not block bridged traffic.

## Host-Specific Stack Notes

- **`unraid-nas` (`192.168.1.95`)**:
  - `jenkins-docker-agent` (`stacks/unraid-nas/buildbox/compose.yaml`) mounts
    the host Docker socket, spawning intermediate `docker build` containers
    directly on the NAS host.
  - `nas-reverse-proxy` (`stacks/unraid-nas/reverse-proxy/compose.yaml`) runs
    `nginx-proxy-manager` (admin API on port `81`, SQLite DB persisted on the
    host at `/mnt/user/docker/nginx-proxy-manager/data/database.sqlite`),
    `cloudflare-ddns`, `crowdsec`, `crowdsec-web-ui`, and `cs-unifi-bouncer`
    (which syncs CrowdSec decisions as firewall groups to the UniFi Network
    controller at `https://192.168.1.181:443`). When `crowdsec` restarts or
    updates, its Local API (LAPI on port `8080`) takes ~9 seconds to
    initialize, causing `cs-unifi-bouncer`
    (`http://crowdsec:8080/v1/decisions/stream`) to briefly restart-loop and
    trigger a transient Komodo stack unhealthy alert.
  - `nas-ai` (`stacks/unraid-nas/ai/compose.yaml`) mounts
    `/mnt/user/media/Reels:/reels` for downloaded Instagram Reels storage and
    `/mnt/user/docker/Instagram:/instagram` for `cookies.txt` access inside the
    Antigravity container.
- **`unraid-seedbox` (`192.168.1.46`)**: `seedbox-torrent` runs `qbittorrent`,
  `qbittorrent-private`, `qui` (healthcheck on `localhost:7476`), `mousetrap`
  (healthcheck on `127.0.0.1:39842`), and `upload-assistant`.
- **`unraid-aibox`**:
  - Runs `aibox-ai` (including `Strata`) and `aibox-gaming`. In `aibox-gaming`,
    `wolf` and `wolf-den` share `/var/run/wolf/wolf.sock`; `wolf-den` requires
    `/var/run/wolf:rw` and write permissions on `wolf.sock` for its ASP.NET
    Core process to connect to the Wolf API.
  - When syncing `unraid-aibox` backups via `rsync-backup` to
    `192.168.1.72:/mnt/external_hdd/backup/aibox/` as `jklinker`, pass
    `--omit-dir-times` (`-O`) if the destination top-level directory is owned
    by `root:root` to prevent non-zero exit codes from directory timestamp
    warnings.
- **`pi-studio` (`192.168.1.72`, Boulder)**:
  - Runs on a 13.8 GiB SD card root filesystem with `/mnt/external_hdd`, has no
    site-to-site WireGuard route to Estes LAN devices, and connects to
    `komodo-core` on `unraid-nas` via an outbound Periphery-to-Core WebSocket
    connection polled hourly (`KOMODO_RESOURCE_POLL_INTERVAL: 1-hr`).
  - Hosts `pi-studio-dashboard` (`uptime-kuma` at
    `https://uptime.klinker.computer` with SQLite DB at
    `/home/jklinker/uptime/kuma.db`, and `glances`), `pi-studio-reverse-proxy`
    (`nginx-proxy-manager` / `openresty`, `cloudflare-ddns`, `crowdsec`,
    `cs-unifi-bouncer`, and `crowdsec-web-ui`), and `pi-studio-files`
    (`filebrowser-quantum` at `https://files.klinker.computer` and
    `rsync-backup`).
  - Uptime Kuma includes an `Estes Battery` monitor (ID `80`, under the
    `Server` group and `all` status page) that polls
    `https://home-api.klinker-cabin.computer/api/solar` every 120s (48s
    timeout, 2 retries) via an `HTTP(s) - Json Query` monitor to alert Slack
    when the Estes battery drops below 50%.
  - `filebrowser-quantum` (`/home/jklinker/filebrowser-quantum/data/config.yaml`
    with `/mnt/external_hdd`) configures sources (`/srv/home` and `/srv/backup`)
    with `rules: [{ folderPath: "/", viewable: true }]` so directories are
    browsed on-demand (`GetFsInfoViewableOnly`) without generating a
    container-layer SQLite index (`/home/filebrowser/tmp/sql/index_all.db`) or
    waking up the spun-down `/mnt/external_hdd` drive.

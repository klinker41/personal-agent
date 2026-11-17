---
topic: pi-lab
category: project
tags: [project, pi-lab]
updated_at: 2026-11-17T07:00:49.037153+00:00
confidence: 0.95
---

# Project: Pi-Lab

- Cluster configuration is maintained across both
`/mnt/user/docker/ansible-estes` and `/mnt/user/docker/ansible-boulder`
(`setup-nas.yml` and `setup-docker.yml`).
- `pi-lab-nas` bridges cluster nodes over `br0` using `systemd-networkd`
(`ForwardDelaySec=0`, `STP=no`, `RequiredFamilyForOnline=ipv4`) and drives a DSI
status screen via `pilab-monitor.service` (`/opt/pilab_monitor/monitor.py`),
which polls every 5s until all 9 nodes respond and every 300s thereafter.

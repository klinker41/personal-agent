---
topic: docker-bridge-iptables
category: knowledge
tags: [knowledge, docker-bridge-iptables]
updated_at: 2026-10-26T06:02:20.265387+00:00
confidence: 0.95
---

# Knowledge: Docker-Bridge-Iptables

- Docker loads the `br_netfilter` kernel module
(`net.bridge.bridge-nf-call-iptables = 1`) and sets the default `iptables`
`FORWARD` chain policy to `DROP` , causing Layer-2 frames traversing host Linux
bridges (e.g., `br0` ) to be dropped unless explicit
`iptables -I FORWARD -i <bridge> -j ACCEPT` and `-o <bridge> -j ACCEPT` rules
are added.
- When UFW is disabled (`ufw: state: disabled`), UFW route rules do not insert
`iptables` `FORWARD` entries; a systemd unit bound to `docker.service`
(`PartOf=docker.service` and `WantedBy=multi-user.target docker.service` )
ensures custom bridge `FORWARD ACCEPT` rules persist across both reboots and
Docker daemon restarts.

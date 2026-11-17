---
topic: docker-swarm-recovery
category: knowledge
tags: [knowledge, docker-swarm-recovery]
updated_at: 2026-11-17T07:00:49.038969+00:00
confidence: 0.95
---

# Knowledge: Docker-Swarm-Recovery

- When Docker Swarm node TLS certificates (`swarm-node.crt`) expire, `dockerd`
fails to initialize Swarm mode after a host reboot and stops starting Swarm
services, while `docker info --format '{{.Swarm.LocalNodeState}}'` reports
`error` instead of `inactive` or `active`.
- Ansible playbooks that initialize or reset Docker Swarm should check
`LocalNodeState != 'active'` rather than only checking for `'inactive'`, and
run `docker swarm leave --force` whenever `LocalNodeState` is neither `active`
nor `inactive` (such as `error`) prior to executing `docker swarm init` or
`docker swarm join`.

- During cold boot on dual-stack networks, IPv6 Router Advertisements can
trigger `network-online.target` long before DHCPv4 completes, causing
`docker.service` to fail Swarm manager UDP dials
(`connect: network is unreachable`), exhaust `StartLimitBurst=3`, and stay
stopped even after IPv4 is assigned; require IPv4 connectivity before starting
`docker.service`.
- `systemd-networkd` bridges default to a 15-second `ForwardDelay` and consider
IPv6 link-local sufficient for online state; set `ForwardDelaySec=0` and
`STP=no` in `.netdev` plus `RequiredFamilyForOnline=ipv4` in `.network` to
prevent downstream DHCPv4 delays and premature `systemd-networkd-wait-online`
completion.

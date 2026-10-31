---
topic: docker-swarm-recovery
category: knowledge
tags: [knowledge, docker-swarm-recovery]
updated_at: 2026-10-26T06:01:57.294660+00:00
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

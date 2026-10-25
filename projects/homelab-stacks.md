---
topic: homelab-stacks
category: projects
tags: [komodo, gitops, docker, stacks, homelab]
updated_at: 2026-10-25T06:01:41.566792+00:00
confidence: 1.0
---

# Homelab-Stacks

- GitOps repository at `/workspace/homelab-stacks` (`klinker41/homelab-stacks`)
driving all 34 Komodo v2 stacks across 6 environments (`unraid-nas`,
`unraid-seedbox` , `unraid-aibox` , `pi-studio` , `pi-lab` , `pi-swarm` ).
- Compose files live in `stacks/<host>/<stack>/compose.yaml` using `${VAR_NAME}`
variable substitution, while `syncs/homelab.toml` maps stack `environment`
entries to encrypted/masked Komodo `Variable` resources via
`VAR_NAME=[[KOMODO_SECRET_NAME]]` .
- Pushing to `main` triggers the `gitops-deploy` Komodo Procedure
(`https://komodo.klinker-cabin.computer/listener/github/procedure/gitops-deploy`),
which runs `RunSync` (`homelab-sync`) followed by `BatchDeployStackIfChanged`
(`pattern = "*"`).

- Komodo (`komodo-core` on `unraid-nas` ) manages remote Docker Swarms
(`pi-swarm`, `pi-lab` ) and `unraid-aibox` stacks (`aibox-ai`, including
`Strata` ), relying on `gluetun-home` and `gluetun-home-backup` WireGuard
site-to-site VPN containers for cross-site connectivity.

- Komodo Core (`komodo-core`, `komodo-mongo` , `komodo-periphery` ) runs on
`unRAID NAS` and manages servers (`unraid-nas`, `unraid-seedbox` ) and swarms
(`pi-swarm`, `pi-lab` ) via declarative Resource Sync.

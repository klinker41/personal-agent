---
topic: strata-container-config
category: knowledge
tags: [knowledge, strata-container-config]
updated_at: 2026-11-22T07:17:23.104015+00:00
confidence: 0.95
---

# Knowledge: Strata-Container-Config

- Setting REINSTALL=1 on the Strata container forces docker-entrypoint.sh to
regenerate /data/config/strata-<family>-<model>.json on startup so updated
environment variables (such as CONTEXT and KV) overwrite stale persisted config
files without re-downloading cached models or packs.
- Running Qwen3.8-Flash-Next Coder in Strata with FAMILY=coder, MODEL=IQ1_M,
CONTEXT=131072, and KV=k8v4 operates in low-RAM hybrid mode (~13 GB system RAM
with ~43% of experts offloaded to GPU), fitting comfortably on a 32 GB RAM host.

- Upstream Niko1221/Strata moved its source-build Dockerfile to
tools/docker/Dockerfile.src and requires --build-arg STRATA_COMMIT=$(git
rev-parse HEAD) because .git is excluded from the build context; the
jklinker/strata-no-vision-ada image builds with --build-arg
CUDA_ARCHITECTURES=89 (Ada/RTX 40) and --build-arg BUILD_VISION=0.

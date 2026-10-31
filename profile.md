---
topic: profile
category: identity
tags: [user, preferences, conventions]
updated_at: 2026-10-30T14:35:00+00:00
compacted_at: 2026-10-30T14:35:00+00:00
confidence: 1.0
---

# User Profile & Development Preferences

This document records core personal habits, development preferences, and
global conventions across projects.

## Core Preferences
- **Language & Styling:** Clean, readable, idiomatic code with type annotations
  in Python and strict TypeScript.
- **Architecture:** Prefer simple, self-contained implementations and shared
  utilities over unnecessary external dependencies.
- **Service Architecture:** Prefer consolidated single-container Bun/Hono or
  Node.js architectures with filesystem-based JSON storage and in-process task
  queues over multi-container Python/PostgreSQL/Redis/Celery setups for homelab
  media generators.
- **Configuration Defaults:** Define application and model configuration
  defaults in code (e.g. `models.ts`) rather than baking default values into
  Dockerfiles.
- **Root Cause Remediation:** Prefer correcting underlying data and permissions
  on disk rather than introducing permissive fallback logic or compatibility
  workarounds in application code.
- **Code Reviews & Media Workflow:** Prefer code reviews to be performed by a
  higher-capability model tier (`pro`), and use a draft-and-refine workflow for
  media generation (generating rapid 360p drafts, refining prompts/shots, then
  upscaling final output).

## Communication & Operational Style
- Concise, direct, and action-oriented responses with transparent rationales.
- Post Slack bot replies in direct messages (DMs) inline at the top level
  rather than in threads, while using Slack as the centralized notification hub
  for self-hosted services and personal agent alerts.
- Follow a cautious "diagnose and ask first" workflow for automated homelab
  alert remediation before executing container restarts or destructive actions.

## Homelab Infrastructure
- **Unraid Hosts:** Operates three Unraid OS servers (`unraid-nas` at
  `192.168.1.95` powered by a 20-thread Intel Core Ultra 7 265K,
  `unraid-seedbox`, and `unraid-aibox`), using `PUID=99` and `PGID=100`
  (`nobody:users`), workspace paths under `/mnt/user/home/jklinker/Coding`,
  container data under `/mnt/user/docker/`, and Docker Hub user `jklinker`.
- **Container & Cluster Orchestration:** Manages all 34 stacks across the three
  Unraid servers, `pi-studio` (in Boulder, hosting Uptime Kuma), and Raspberry
  Pi Docker Swarm clusters (`pi-swarm` and `pi-lab` in Estes, including
  `pi-lab-nas` at `192.168.1.113` with a Pod Bay 3 ATX Power Management HAT)
  via Komodo v2 (`komodo-core` on `unraid-nas` and `komodo-periphery` agents)
  and Ansible Semaphore (`semaphore-estes` and `semaphore-boulder`).
- **Security & Access:** Prefer container-dedicated SSH keys and host SSH
  access over mounting host credentials or `/var/run/docker.sock` into
  containers.
- **Media, Sensors & Display:** Runs Jellyfin 12, Sonarr, Radarr, qBittorrent,
  and Tdarr; monitors Emporia Vue energy, Airthings air quality/radon (Estes
  and Boulder), and Tesla solar/Powerwall via Teslemetry; and displays homelab
  status on a TRMNL 800x480 e-ink display refreshed every 15 minutes.

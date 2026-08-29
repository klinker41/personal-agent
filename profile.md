---
topic: profile
category: identity
tags: [user, preferences, conventions]
updated_at: 2026-08-29T11:57:01.235251+00:00
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

## Communication Style
- Concise, direct, and action-oriented.
- Focus on practical implementations with transparent rationales.

- Prefers a draft-and-refine workflow for media generation (generating rapid
low-res/360p drafts, tweaking prompts and shots manually, then upscaling final
output).

- Prefers code reviews to be performed by a different or higher-capability model
tier than the model used for writing code.

- Prefers container-dedicated SSH keys and Git configurations over mounting host
credentials into Docker containers.
- Runs an Unraid NAS homelab environment with workspace paths under
`/mnt/user/home/jklinker/Coding` and container data in `/mnt/user/docker/`.

- Docker Hub username is `jklinker`.

- Homelab server runs Unraid NAS, using PUID 99 and PGID 100 (nobody:users) for
container permissions.

- Uses a TRMNL 800x480 e-ink display refreshed every 15 minutes to monitor
homelab status.
- Homelab environment includes unRAID, Portainer across 3 environments
('Server', 'Seedbox', 'AIbox'), Sonarr, Radarr, qBittorrent, Tdarr, and custom
home sensors (Energy, Radon, Solar across Boulder and Estes Park locations).

- Runs homelab infrastructure and containers on Unraid OS.
- Prefers host SSH access over mounting /var/run/docker.sock into containers for
security.

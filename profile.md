---
topic: profile
category: identity
tags: [user, preferences, conventions]
updated_at: 2026-11-23T07:32:03.200646+00:00
compacted_at: 2026-11-23T07:32:03.200646+00:00
confidence: 1.0
---

# User Profile & Development Preferences

## Core Preferences
- **Language & Styling:** Clean, readable, idiomatic code with Python type
  annotations and strict TypeScript.
- **Architecture & Dependencies:** Prefer simple, self-contained
  implementations and shared utilities over unnecessary external
  dependencies. For homelab media generators, prefer consolidated
  single-container Bun/Hono or Node.js services with filesystem-based JSON
  storage and in-process task queues over multi-container
  Python/PostgreSQL/Redis/Celery setups. Migrate client code for breaking
  API changes immediately during dependency updates rather than deferring
  them so projects stay close to head, sending notifications whenever code
  changes are required.
- **Configuration & Remediation:** Define application and model
  configuration defaults in code (e.g., `models.ts`) rather than baking
  them into Dockerfiles. Fix underlying data and permissions on disk rather
  than introducing permissive fallback logic or compatibility workarounds
  in application code.
- **Model Tiering & Agent Efficiency:** Right-size recurring agent sidecars
  and workflows (`flash_lite` for routine homelab/media cleanup, `flash`
  for interactive Slack chat and host updates, and `pro` for code reviews
  and weekly meta-audits). Handle deterministic tasks in Python pre-filters
  without spawning LLM turns.
- **Media Workflow:** Use a draft-and-refine workflow for media generation:
  generate rapid 360p drafts, refine prompts/shots, then upscale final
  output.
- **Android & Mobile:** Uses a Pixel 11 Pro Fold for daily use and Android
  testing, prioritizing UI and scroll performance in both unfolded and
  folded display states.

## Communication & Operational Style
- **Response Style:** Concise, direct, and action-oriented responses with
  transparent rationales.
- **Slack Conventions:** Use Slack as the centralized notification hub for
  self-hosted services and personal agent alerts; post bot replies in
  direct messages (DMs) inline at the top level rather than in threads.
  Keep scheduled background sync/maintenance sidecars and homelab tasks
  silent on clean, routine green runs, alerting only on actionable
  problems, failures, or blockers.
- **Autonomous Remediation:** Perform non-destructive remediation
  immediately without asking during automated homelab alert investigations
  (e.g., restarting containers/stacks or fixing issues via MCP or SSH), and
  ask for confirmation only before destructive actions.

## Homelab Infrastructure
- **Unraid Hosts:** Operates three Unraid OS servers (`unraid-nas` at
  `192.168.1.95` with a 20-thread Intel Core Ultra 7 265K,
  `unraid-seedbox`, and `unraid-aibox`), using `PUID=99` and `PGID=100`
  (`nobody:users`), workspace paths under `/mnt/user/home/jklinker/Coding`,
  container data under `/mnt/user/docker/`, and Docker Hub user `jklinker`.
  `unraid-nas` is being upgraded to 96GB of RAM, making strict Docker
  container and Gradle build memory caps unnecessary.
- **Orchestration:** Manages all 34 stacks across the three Unraid servers,
  `pi-studio` (in Boulder, hosting Uptime Kuma), and Estes Raspberry Pi
  Docker Swarm clusters (`pi-swarm` and `pi-lab`, including `pi-lab-nas` at
  `192.168.1.113` with a Pod Bay 3 ATX Power Management HAT) via Komodo v2
  (`komodo-core` on `unraid-nas` and `komodo-periphery` agents) and Ansible
  Semaphore (`semaphore-estes` and `semaphore-boulder`).
- **CI/CD:** Runs Jenkins on `unraid-nas` with jobs manually configured via
  the Jenkins UI (without Jenkinsfiles) and build failure alerts sent via
  the Jenkins Slack Notification plugin.
- **Security & Access:** Prefer container-dedicated SSH keys and host SSH
  access over mounting host credentials or `/var/run/docker.sock` into
  containers.
- **Media, Sensors & Display:** Runs Jellyfin 12, Sonarr, Radarr,
  qBittorrent, and Tdarr; monitors Emporia Vue energy, Airthings air
  quality/radon (Estes and Boulder), Tesla solar/Powerwall via Teslemetry,
  and a Growatt SPF 3500 inverter (Lithium mode, connected to `unraid-nas`
  at `/dev/ttyUSB0` via USB serial Modbus RTU); and displays homelab status
  on a TRMNL 800x480 e-ink display refreshed every 15 minutes.

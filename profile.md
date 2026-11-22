---
topic: profile
category: identity
tags: [user, preferences, conventions]
updated_at: 2026-11-22T07:21:12.557135+00:00
compacted_at: 2026-11-21T07:31:41.723099+00:00
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
  Python/PostgreSQL/Redis/Celery setups.
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

## Communication & Operational Style
- **Response Style:** Concise, direct, and action-oriented responses with
  transparent rationales.
- **Slack Conventions:** Use Slack as the centralized notification hub for
  self-hosted services and personal agent alerts; post bot replies in
  direct messages (DMs) inline at the top level rather than in threads.
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
- **Orchestration:** Manages all 34 stacks across the three Unraid servers,
  `pi-studio` (in Boulder, hosting Uptime Kuma), and Estes Raspberry Pi
  Docker Swarm clusters (`pi-swarm` and `pi-lab`, including `pi-lab-nas` at
  `192.168.1.113` with a Pod Bay 3 ATX Power Management HAT) via Komodo v2
  (`komodo-core` on `unraid-nas` and `komodo-periphery` agents) and Ansible
  Semaphore (`semaphore-estes` and `semaphore-boulder`).
- **Security & Access:** Prefer container-dedicated SSH keys and host SSH
  access over mounting host credentials or `/var/run/docker.sock` into
  containers.
- **Media, Sensors & Display:** Runs Jellyfin 12, Sonarr, Radarr,
  qBittorrent, and Tdarr; monitors Emporia Vue energy, Airthings air
  quality/radon (Estes and Boulder), Tesla solar/Powerwall via Teslemetry,
  and a Growatt SPF 3500 inverter (Lithium mode, connected to `unraid-nas`
  at `/dev/ttyUSB0` via USB serial Modbus RTU); and displays homelab status
  on a TRMNL 800x480 e-ink display refreshed every 15 minutes.

- Unraid NAS homelab server (unraid-nas) is being upgraded to 96GB of RAM,
making strict Docker container and Gradle build memory caps unnecessary.

- Runs a Jenkins instance on the nas homelab host with jobs manually configured
via the Jenkins UI (without Jenkinsfiles) and uses the Jenkins Slack
Notification plugin for build failure alerts.

- Prefers scheduled background sync and maintenance sidecars to run silently on
clean/successful runs and notify Slack only on failures or blockers.

- Prefers migrating client code for breaking API changes immediately during
dependency updates rather than deferring them so projects stay close to head,
with notifications sent whenever code changes are required.

- Prefers homelab Slack notifications to alert only on actionable problems and
failures rather than routine green tasks.

---
topic: antigravity-docker
category: project
tags: [project, antigravity-docker]
updated_at: 2026-08-30T11:29:57.596460+00:00
confidence: 0.95
---

# Project: Antigravity-Docker

## Overview & Architecture
- Image published at `jklinker/antigravity-docker:latest` providing a headless
  containerized Google Antigravity (`agy`) environment on port 4400.
- Documentation and configurations remain generic across host environments.
- Enforces non-root execution (`developer` user via `gosu`) with passwordless
  sudo disabled, dynamic `PUID`/`PGID`, and `umask 0002` for permissions across
  `conversations/`, `brain/`, and `annotations/`.
- Replaces `/var/run/docker.sock` mounting with an SSH-based web terminal
  (`ttyd`) for secure host command execution without socket exposure.

## Configuration & Environment
- Environment variables: `RC_NAME`, `AGY_PORT` (default 4400), `AUTH_PASSWORD`,
  `ENABLE_IDE` (default true), `ENABLE_TERMINAL` (default true),
  `HOST_SSH_DIR`, and `BLOCK_TELEMETRY` (default true; sinkholes Google
  telemetry to `0.0.0.0` via `/etc/hosts` and sets OpenTelemetry opt-out
  variables).
- Initial authentication runs inside the container via `setup` subcommand with
  `~/.gemini` mounted.
- Persistent state (`antigravity_state.pbtxt` with `installation_uuid` and
  schema migrations) and logs reside strictly in
  `$GEMINI_DIR/antigravity-cli/` (checked by proxy at `.../cli.log`).
- `entrypoint.sh` initializes individual project configs under
  `$GEMINI_DIR/config/projects/` when empty (no monolithic `projects.json`).
- Default Antigravity settings: `enableTerminalSandbox: true`,
  `autoExecutionPolicy: CASCADE_COMMANDS_AUTO_EXECUTION_PROCEED_IN_SANDBOX`,
  `nonWorkspaceFiles: ALLOW`, with VS Code IDE and Host Terminal sidebar
  buttons.

## Auth Proxy & Gateway (`proxy/auth-proxy.js`)
- Security: Dynamic 256-bit session tokens, in-memory cleanup, IP rate-limiting
  on `/__auth/login`, 16 KB body limit, security headers (CSP,
  `X-Content-Type-Options`, `X-Frame-Options`), path traversal guards, and
  centralized body parsing in `proxy/lib/security.js`.
- UI & Routes: Unauthenticated `/status` health check (`200`/`503`), persistent
  favicon injection (`MutationObserver`), and shared glassmorphic cosmic UI
  with 2D canvas particles (`renderPageLayout`, `BASE_PAGE_CSS`).
- Protocol Handling: Forces `useWebSocket=true` on root and `/c/...` routes;
  sets `X-Accel-Buffering: no`; preserves `TE: trailers`, `Trailer`, and
  `grpc-status`; flushes headers immediately; strips hop-by-hop headers; and
  avoids TCP RST packets on upstream socket cleanup.

## Sidecar Management (`proxy/sidecar-manager.js`)
- Supervises background workers and cron prompts with 5-field parsing, restart
  policies, unified env resolution (`buildSidecarEnv`), and authenticated
  `/sidecars` UI and REST APIs.
- Standalone sidecars: Defined in `~/.gemini/config/sidecars/<id>/sidecar.json`,
  with enabled flags in `~/.gemini/config/config.json` (`sidecars[id].enabled`).
- Plugin sidecars: Discovered in `<plugin>/sidecars/<name>/sidecar.json`,
  namespaced as `<plugin-name>/<sidecar-name>`, executed with isolated `cwd`
  and prepended `PATH`, with `PLUGIN` UI badges and configuration resets.

## Testing
- Tests execute via Node.js native test runner (`node --test tests/*.js`).

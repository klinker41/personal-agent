---
topic: antigravity-docker
category: project
tags: [project, antigravity-docker]
updated_at: 2026-09-02T00:30:54.139907+00:00
confidence: 0.95
---

# Project: Antigravity-Docker

## Overview & Architecture
- Headless containerized Google Antigravity (`agy`) environment
  (`jklinker/antigravity-docker:latest`) running on port 4400.
- Security & Permissions: Enforces non-root execution (`developer` via `gosu`),
  dynamic `PUID`/`PGID`, disabled passwordless sudo, and `umask 0002` across
  `conversations/`, `brain/`, and `annotations/`.
- Host Execution: Replaces `/var/run/docker.sock` with an SSH-based web terminal
  (`ttyd`) for secure host command execution without socket exposure.

## Configuration & Environment
- Environment Variables: `RC_NAME`, `AGY_PORT` (default 4400), `AGY_HUB_PORT`
  (default 4402; passed via `--hub-port` to `agy --remote-control` for
  deterministic upstream connections without log scraping), `AUTH_PASSWORD`,
  `ENABLE_IDE` (default true), `ENABLE_TERMINAL` (default true),
  `HOST_SSH_DIR`, and `BLOCK_TELEMETRY` (default true; sinkholes Google
  telemetry to `0.0.0.0` via `/etc/hosts` and sets OpenTelemetry opt-out
  variables).
- Initialization & State: Initial auth configured via `setup` subcommand with
  `~/.gemini` mounted. `entrypoint.sh` populates `$GEMINI_DIR/config/projects/`
  when empty and purges stale candidate CSRF token files on daemon startup.
  Persistent state (`antigravity_state.pbtxt` with `installation_uuid` / schema
  migrations) and logs (`cli.log`) reside in `$GEMINI_DIR/antigravity-cli/`.
- Default Settings: `enableTerminalSandbox: true`, `autoExecutionPolicy:
  CASCADE_COMMANDS_AUTO_EXECUTION_PROCEED_IN_SANDBOX`, `nonWorkspaceFiles:
  ALLOW`, with VS Code IDE and Host Terminal sidebar buttons.

## Auth Proxy & Gateway (`proxy/auth-proxy.js`)
- Security & Middleware: Dynamic 256-bit session tokens, in-memory cleanup, IP
  rate-limiting on `/__auth/login`, 16 KB body limit, path traversal guards,
  security headers (CSP, `X-Content-Type-Options`, `X-Frame-Options`), and
  centralized body parsing in `proxy/lib/security.js`.
- UI & Routes: Unauthenticated `/status` health check (`200`/`503`), persistent
  favicon injection (`MutationObserver`), and shared glassmorphic cosmic UI
  with 2D canvas particles (`renderPageLayout`, `BASE_PAGE_CSS`).
- Protocol Handling: Forces `useWebSocket=true` on root and `/c/...` routes,
  sets `X-Accel-Buffering: no`, flushes headers immediately, strips
  hop-by-hop headers, preserves `TE: trailers`, `Trailer`, and `grpc-status`,
  and avoids TCP RST packets on upstream socket cleanup.

## Sidecar Management (`proxy/sidecar-manager.js`)
- Supervision & Lifecycle: Manages background workers and 5-field cron prompts
  with restart policies, unified env resolution (`buildSidecarEnv`), queries
  `127.0.0.1:${AGY_HUB_PORT}` for live CSRF tokens, awaits upstream Language
  Server readiness via `waitForUpstream()`, and provides authenticated
  `/sidecars` UI and REST APIs.
- Standalone Sidecars: Defined in `~/.gemini/config/sidecars/<id>/sidecar.json`
  and toggled via `sidecars[id].enabled` in `~/.gemini/config/config.json`.
- Plugin Sidecars: Discovered at `<plugin>/sidecars/<name>/sidecar.json`,
  namespaced as `<plugin-name>/<sidecar-name>`, run with isolated `cwd` and
  prepended `PATH`, featuring `PLUGIN` UI badges and configuration resets.

## Testing & Quality
- Test Runner: Native Node.js test runner (`node --test tests/*.js`).
- Test Isolation: `tests/test-sidecar-manager.js` isolates test state by
  restoring mock environment variables and disk files in `finally` blocks to
  prevent runtime CSRF token pollution.

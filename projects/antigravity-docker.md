---
topic: antigravity-docker
category: project
tags: [project, antigravity-docker]
updated_at: 2026-09-03T00:31:02.749739+00:00
confidence: 0.95
---

# Project: Antigravity-Docker

## Overview & Architecture
- Headless containerized Google Antigravity (`agy`) runtime
  (`jklinker/antigravity-docker:latest`) serving on port 4400.
- Security & Permissions: Non-root execution (`developer` via `gosu`),
  dynamic `PUID`/`PGID`, disabled passwordless sudo, and `umask 0002` across
  `conversations/`, `brain/`, and `annotations/`.
- Host Access: Replaces `/var/run/docker.sock` exposure with an isolated
  SSH-based web terminal (`ttyd`) for secure host execution.

## Configuration & Environment
- Environment Variables: `RC_NAME`, `AGY_PORT` (default 4400), `AGY_HUB_PORT`
  (default 4402; passed via `--hub-port` to `agy --remote-control` for
  deterministic upstream connections without log scraping), `AUTH_PASSWORD`,
  `ENABLE_IDE` (default true), `ENABLE_TERMINAL` (default true),
  `HOST_SSH_DIR`, and `BLOCK_TELEMETRY` (default true; sinkholes Google
  telemetry to `0.0.0.0` via `/etc/hosts` and sets OpenTelemetry opt-out
  variables).
- Initialization & State: Initial auth configured via `setup` subcommand with
  `~/.gemini` mounted. `entrypoint.sh` populates
  `$GEMINI_DIR/config/projects/` when empty and purges stale candidate CSRF
  tokens on daemon startup. Persistent state (`antigravity_state.pbtxt` with
  `installation_uuid` and schema migrations) and logs (`cli.log`) reside in
  `$GEMINI_DIR/antigravity-cli/`.
- Default Settings: `enableTerminalSandbox: true`, `autoExecutionPolicy:
  CASCADE_COMMANDS_AUTO_EXECUTION_PROCEED_IN_SANDBOX`, `nonWorkspaceFiles:
  ALLOW`, with sidebar shortcuts for VS Code IDE and Host Terminal.

## Auth Proxy & Gateway (`proxy/auth-proxy.js`)
- Security & Middleware: Dynamic 256-bit session tokens, in-memory cleanup,
  IP rate-limiting on `/__auth/login`, 16 KB body limit, path-traversal guards,
  security headers (CSP, `X-Content-Type-Options`, `X-Frame-Options`), and
  centralized body parsing in `proxy/lib/security.js`.
- Protocol Handling: Enforces `useWebSocket=true` on `/` and `/c/...`, sets
  `X-Accel-Buffering: no`, flushes headers immediately, strips hop-by-hop
  headers, preserves `TE: trailers`, `Trailer`, and `grpc-status`, and avoids
  TCP RST packets on upstream socket teardown.
- UI & Routes: Unauthenticated `/status` health check (`200`/`503`), persistent
  favicon injection (`MutationObserver`), and shared cosmic glassmorphic UI
  with 2D canvas particles (`renderPageLayout`, `BASE_PAGE_CSS`).

## Sidecar Management (`proxy/sidecar-manager.js`)
- Supervision & Lifecycle: Manages background workers and 5-field cron tasks
  with restart policies, unified env resolution (`buildSidecarEnv`), live CSRF
  token querying at `127.0.0.1:${AGY_HUB_PORT}`, Language Server readiness
  polling (`waitForUpstream()`), and authenticated `/sidecars` REST APIs and
  management UI.
- Standalone Sidecars: Defined in `~/.gemini/config/sidecars/<id>/sidecar.json`
  and toggled via `sidecars[id].enabled` in `~/.gemini/config/config.json`.
- Plugin Sidecars: Discovered at `<plugin>/sidecars/<name>/sidecar.json`
  (namespaced as `<plugin-name>/<sidecar-name>`), run with isolated `cwd` and
  prepended `PATH`, featuring `PLUGIN` UI badges and configuration resets.

## Testing & Quality
- Test Runner: Native Node.js test runner (`node --test tests/*.js`).
- Test Isolation: `tests/test-sidecar-manager.js` isolates state by restoring
  mock environment variables and filesystem fixtures in `finally` blocks to
  prevent CSRF token pollution.

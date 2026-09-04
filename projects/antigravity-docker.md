---
topic: antigravity-docker
category: project
tags: [project, antigravity-docker]
updated_at: 2026-09-04T00:30:48.455071+00:00
confidence: 0.95
---

# Project: Antigravity-Docker

## Overview & Architecture
- Headless containerized Google Antigravity (`agy`) runtime
  (`jklinker/antigravity-docker:latest`) serving on port 4400.
- Security & Permissions: Non-root execution (`developer` via `gosu`), dynamic
  `PUID`/`PGID`, disabled passwordless sudo, and `umask 0002` across
  `conversations/`, `brain/`, and `annotations/`.
- Host Access: Replaces `/var/run/docker.sock` exposure with an isolated
  SSH-based web terminal (`ttyd`) for secure host execution.

## Configuration & Environment
- Environment Variables:
  - Core & Auth: `RC_NAME`, `AUTH_PASSWORD`, `HOST_SSH_DIR`.
  - Feature Flags: `ENABLE_IDE` (default true), `ENABLE_TERMINAL` (default
    true).
  - Networking: `AGY_PORT` (default 4400); `AGY_HUB_PORT` (default 4402;
    passed via `--hub-port` to `agy --remote-control` for deterministic
    upstream connections without log scraping).
  - Telemetry: `BLOCK_TELEMETRY` (default true; sinkholes Google telemetry to
    `0.0.0.0` via `/etc/hosts` and sets OpenTelemetry opt-out variables).
- Initialization & State: Initial auth configured via `setup` subcommand with
  `~/.gemini` mounted. On startup, `entrypoint.sh` populates empty
  `$GEMINI_DIR/config/projects/` and purges stale candidate CSRF tokens.
  Persistent state (`antigravity_state.pbtxt` with `installation_uuid` and
  schema migrations) and logs (`cli.log`) reside in
  `$GEMINI_DIR/antigravity-cli/`.
- Default Settings: `enableTerminalSandbox: true`, `autoExecutionPolicy:
  CASCADE_COMMANDS_AUTO_EXECUTION_PROCEED_IN_SANDBOX`, `nonWorkspaceFiles:
  ALLOW`, and sidebar shortcuts for VS Code IDE and Host Terminal.

## Auth Proxy & Gateway (`proxy/auth-proxy.js`)
- Security & Middleware: Dynamic 256-bit session tokens, in-memory session
  cleanup, IP rate-limiting on `/__auth/login`, 16 KB body limit, path-traversal
  guards, security headers (CSP, `X-Content-Type-Options`, `X-Frame-Options`),
  and centralized body parsing in `proxy/lib/security.js`.
- Protocol Handling: Enforces `useWebSocket=true` on `/` and `/c/...`, sets
  `X-Accel-Buffering: no`, flushes headers immediately, strips hop-by-hop
  headers, preserves `TE: trailers`, `Trailer`, and `grpc-status`, and avoids
  TCP RST packets on upstream socket teardown.
- UI & Routes: Unauthenticated `/status` health check (`200`/`503`), persistent
  favicon injection (`MutationObserver`), and shared cosmic glassmorphic UI
  with 2D canvas particles (`renderPageLayout`, `BASE_PAGE_CSS`).

## Sidecar Management (`proxy/sidecar-manager.js`)
- Supervision & Lifecycle: Manages background workers and 5-field cron tasks
  via authenticated `/sidecars` REST APIs and UI. Features restart policies,
  unified env resolution (`buildSidecarEnv`), upstream Language Server polling
  (`waitForUpstream()`), and live CSRF token querying at
  `127.0.0.1:${AGY_HUB_PORT}`.
- Discovery & Types:
  - Standalone: Defined in `~/.gemini/config/sidecars/<id>/sidecar.json` and
    toggled via `sidecars[id].enabled` in `~/.gemini/config/config.json`.
  - Plugin: Discovered at `<plugin>/sidecars/<name>/sidecar.json` (namespaced
    as `<plugin-name>/<sidecar-name>`); runs with isolated `cwd`, prepended
    `PATH`, `PLUGIN` badges, and configuration resets.

## Testing & Quality
- Test Runner: Native Node.js test runner (`node --test tests/*.js`).
- State Isolation: `tests/test-sidecar-manager.js` isolates state by restoring
  mock environment variables and filesystem fixtures in `finally` blocks to
  prevent CSRF token pollution.

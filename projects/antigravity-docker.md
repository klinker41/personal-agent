---
topic: antigravity-docker
category: project
tags: [project, antigravity-docker]
updated_at: 2026-09-09T00:30:56.670714+00:00
confidence: 0.95
---

# Project: Antigravity-Docker

## Overview & Architecture
- **Runtime & Security:** Headless containerized Google Antigravity (`agy`)
  runtime (`jklinker/antigravity-docker:latest`) running non-root (`developer`
  via `gosu`) with dynamic `PUID`/`PGID`, disabled passwordless sudo, and
  `umask 0002` for `conversations/`, `brain/`, and `annotations/`.
- **Host Execution:** Avoids mounting `/var/run/docker.sock` by providing an
  isolated SSH-based web terminal (`ttyd`) for host commands.

## Configuration & Runtime Environment
- **Networking & Ports:** Defaults to `AGY_PORT=4400` and `AGY_HUB_PORT=4402`
  (passed via `--hub-port` to `agy --remote-control` for deterministic discovery
  without scraping logs).
- **Environment Flags:**
  - Auth & Access: `RC_NAME`, `AUTH_PASSWORD`, `HOST_SSH_DIR`.
  - Feature Toggles: `ENABLE_IDE`, `ENABLE_TERMINAL` (both default `true`).
  - Telemetry: `BLOCK_TELEMETRY` (default `true`; sinkholes telemetry to
    `0.0.0.0` via `/etc/hosts` and sets OpenTelemetry opt-out variables).
- **Lifecycle & Storage:**
  - Initial authentication via `setup` subcommand with mounted `~/.gemini`.
  - `entrypoint.sh` initializes empty `$GEMINI_DIR/config/projects/` and purges
    stale candidate CSRF tokens.
  - Runtime state (`antigravity_state.pbtxt`, `installation_uuid`, migrations)
    and logs (`cli.log`) reside in `$GEMINI_DIR/antigravity-cli/`.
- **Default Policy:** `enableTerminalSandbox: true`, `nonWorkspaceFiles: ALLOW`,
  `autoExecutionPolicy: CASCADE_COMMANDS_AUTO_EXECUTION_PROCEED_IN_SANDBOX`,
  with sidebar navigation shortcuts for VS Code IDE and Host Terminal.

## Auth Proxy & Gateway (`proxy/auth-proxy.js`)
- **Security & Middleware:** Enforces 256-bit session tokens, in-memory session
  cleanup, IP rate-limiting on `/__auth/login`, 16 KB body limit, path
  traversal protections, security headers (CSP, frame/content-type options),
  and centralized body parsing (`proxy/lib/security.js`).
- **Protocol & Streaming:** Enforces `useWebSocket=true` on `/` and `/c/...`,
  disables proxy buffering (`X-Accel-Buffering: no`), flushes headers
  immediately, strips hop-by-hop headers, preserves gRPC streaming headers
  (`TE: trailers`, `Trailer`, `grpc-status`), and suppresses TCP RST during
  upstream socket teardown.
- **Endpoints & UI:** Serves unauthenticated `/status` health checks (`200`/
  `503`), injects dynamic favicons via `MutationObserver`, and provides a
  cosmic glassmorphic UI with 2D canvas particles (`renderPageLayout`,
  `BASE_PAGE_CSS`).

## Sidecar Management (`proxy/sidecar-manager.js`)
- **Supervisor & Engine:** Manages background workers and 5-field cron tasks
  via authenticated `/sidecars` REST APIs/UI. Handles restart policies, unified
  environment variables (`buildSidecarEnv`), Language Server polling
  (`waitForUpstream()`), and live CSRF querying at `127.0.0.1:${AGY_HUB_PORT}`.
- **Sidecar Types:**
  - *Standalone:* Defined in `~/.gemini/config/sidecars/<id>/sidecar.json`;
    toggled via `sidecars[id].enabled` in `~/.gemini/config/config.json`.
  - *Plugin:* Loaded from `<plugin>/sidecars/<name>/sidecar.json`, namespaced
    as `<plugin-name>/<sidecar-name>`, executed with isolated `cwd`,
    prepended `PATH`, `PLUGIN` UI badges, and isolated config resets.

## Testing & Quality
- **Test Suite:** Native Node test runner (`node --test tests/*.js`). State
  isolation in `tests/test-sidecar-manager.js` cleans up mock environment
  variables and filesystem fixtures in `finally` blocks to prevent CSRF token
  leakage across test suites.

---
topic: antigravity-docker
category: project
tags: [project, antigravity-docker]
updated_at: 2026-09-10T00:31:10.175323+00:00
confidence: 0.95
---

# Project: Antigravity-Docker

## Overview & Architecture
- **Runtime & Host Isolation:** Containerized headless Google Antigravity
  (`jklinker/antigravity-docker:latest`) running non-root (`developer` via
  `gosu`) with dynamic `PUID`/`PGID`, disabled passwordless sudo, and
  `umask 0002` for `conversations/`, `brain/`, and `annotations/`. Uses an
  isolated SSH web terminal (`ttyd`) for host commands instead of mounting
  `/var/run/docker.sock`.

## Configuration & Runtime Environment
- **Networking & Ports:** Defaults to `AGY_PORT=4400` and `AGY_HUB_PORT=4402`
  (passed to `agy --remote-control --hub-port` for deterministic discovery
  without log scraping).
- **Environment Flags:**
  - Access & Auth: `RC_NAME`, `AUTH_PASSWORD`, `HOST_SSH_DIR`.
  - Feature Toggles: `ENABLE_IDE`, `ENABLE_TERMINAL` (both default `true`).
  - Telemetry: `BLOCK_TELEMETRY=true` (default; sinkholes telemetry to `0.0.0.0`
    via `/etc/hosts` and sets OpenTelemetry opt-out variables).
- **Storage & Lifecycle:**
  - Initial auth via `setup` subcommand with mounted `~/.gemini`.
  - `entrypoint.sh` initializes empty `$GEMINI_DIR/config/projects/` and purges
    stale candidate CSRF tokens.
  - Runtime state (`antigravity_state.pbtxt`, `installation_uuid`, migrations)
    and `cli.log` reside in `$GEMINI_DIR/antigravity-cli/`.
- **Default Policy & Navigation:** `enableTerminalSandbox: true`,
  `nonWorkspaceFiles: ALLOW`, `autoExecutionPolicy:
  CASCADE_COMMANDS_AUTO_EXECUTION_PROCEED_IN_SANDBOX`, with sidebar navigation
  shortcuts for VS Code IDE and Host Terminal.

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
  `503`), injects dynamic favicons via `MutationObserver`, and renders a cosmic
  glassmorphic UI with 2D canvas particles (`renderPageLayout`,
  `BASE_PAGE_CSS`).

## Sidecar Management (`proxy/sidecar-manager.js`)
- **Supervisor & Engine:** Authenticated `/sidecars` REST APIs/UI manage
  background workers and 5-field cron tasks. Handles restart policies, unified
  environment variables (`buildSidecarEnv`), Language Server polling
  (`waitForUpstream()`), and live CSRF querying at `127.0.0.1:${AGY_HUB_PORT}`.
- **Sidecar Types:**
  - *Standalone:* Defined in `~/.gemini/config/sidecars/<id>/sidecar.json`;
    toggled via `sidecars[id].enabled` in `~/.gemini/config/config.json`.
  - *Plugin:* Loaded from `<plugin>/sidecars/<name>/sidecar.json`, namespaced
    as `<plugin-name>/<sidecar-name>`, executed with isolated `cwd`, prepended
    `PATH`, `PLUGIN` UI badges, and isolated config resets.

## Testing & Quality
- **Test Suite:** Native Node test runner (`node --test tests/*.js`). State
  isolation in `tests/test-sidecar-manager.js` cleans up mock environment
  variables and filesystem fixtures in `finally` blocks to prevent CSRF token
  leakage across test suites.

---
topic: antigravity-docker
category: project
tags: [project, antigravity-docker]
updated_at: 2026-09-11T00:30:40.672833+00:00
confidence: 0.95
---

# Project: Antigravity-Docker

## Architecture & Container Runtime
- **Image & Isolation:** `jklinker/antigravity-docker:latest` runs headless
  as non-root `developer` via `gosu` with dynamic `PUID`/`PGID`, disabled
  passwordless sudo, and `umask 0002` across `conversations/`, `brain/`, and
  `annotations/`.
- **Host Execution:** Employs an isolated SSH web terminal (`ttyd`) rather than
  mounting `/var/run/docker.sock`.

## Configuration & Environment
- **Networking:** Defaults to `AGY_PORT=4400` and `AGY_HUB_PORT=4402` (passed to
  `agy --remote-control --hub-port` for deterministic hub discovery without log
  scraping).
- **Environment Flags:**
  - *Auth & Host:* `RC_NAME`, `AUTH_PASSWORD`, `HOST_SSH_DIR`.
  - *Features:* `ENABLE_IDE`, `ENABLE_TERMINAL` (default: `true`).
  - *Telemetry:* `BLOCK_TELEMETRY=true` (default; sinkholes telemetry to
    `0.0.0.0` via `/etc/hosts` and sets OpenTelemetry opt-out variables).
- **Storage & State:**
  - Initial authentication via `setup` subcommand with mounted `~/.gemini`.
  - `entrypoint.sh` initializes empty `$GEMINI_DIR/config/projects/` and purges
    stale CSRF tokens.
  - Runtime state (`antigravity_state.pbtxt`, `installation_uuid`, migrations)
    and `cli.log` reside in `$GEMINI_DIR/antigravity-cli/`.
- **Default Policies:** `enableTerminalSandbox: true`,
  `nonWorkspaceFiles: ALLOW`, `autoExecutionPolicy:
  CASCADE_COMMANDS_AUTO_EXECUTION_PROCEED_IN_SANDBOX`, with sidebar navigation
  shortcuts for VS Code IDE and Host Terminal.

## Auth Proxy & Gateway (`proxy/auth-proxy.js`)
- **Security:** 256-bit session tokens, in-memory session cleanup, IP
  rate-limiting on `/__auth/login`, 16 KB request body limit, path traversal
  protection, security headers (CSP, frame/content-type options), and
  centralized body parsing via `proxy/lib/security.js`.
- **Protocol & Streaming:** Enforces `useWebSocket=true` for `/` and `/c/...`,
  disables proxy buffering (`X-Accel-Buffering: no`), flushes headers
  immediately, strips hop-by-hop headers, preserves gRPC streaming headers
  (`TE: trailers`, `Trailer`, `grpc-status`), and suppresses upstream TCP RST
  during socket teardown.
- **Endpoints & UI:** Unauthenticated `/status` health checks (`200`/`503`),
  dynamic favicons (`MutationObserver`), and a cosmic glassmorphic UI with 2D
  canvas particles (`renderPageLayout`, `BASE_PAGE_CSS`).

## Sidecar Management (`proxy/sidecar-manager.js`)
- **Engine & Supervisor:** Authenticated `/sidecars` REST API/UI manages
  background workers and 5-field cron tasks. Coordinates restart policies,
  unified env injection (`buildSidecarEnv`), upstream Language Server polling
  (`waitForUpstream()`), and live CSRF token discovery at
  `127.0.0.1:${AGY_HUB_PORT}`.
- **Sidecar Types:**
  - *Standalone:* Defined in `~/.gemini/config/sidecars/<id>/sidecar.json` and
    toggled via `sidecars[id].enabled` in `~/.gemini/config/config.json`.
  - *Plugin:* Defined in `<plugin>/sidecars/<name>/sidecar.json`, namespaced as
    `<plugin-name>/<sidecar-name>`, executed with isolated `cwd`, prepended
    `PATH`, `PLUGIN` badge, and isolated config resets.

## Testing & Quality
- **Test Suite:** Native Node test runner (`node --test tests/*.js`).
- **State Isolation:** `tests/test-sidecar-manager.js` cleans up mock
  environment variables and filesystem fixtures in `finally` blocks to prevent
  CSRF token or state leakage between suites.

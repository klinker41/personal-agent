---
topic: antigravity-docker
category: project
tags: [project, antigravity-docker]
updated_at: 2026-09-08T00:31:25.820470+00:00
confidence: 0.95
---

# Project: Antigravity-Docker

## Overview & Architecture
- **Runtime & Security:** Headless containerized Google Antigravity (`agy`)
  runtime (`jklinker/antigravity-docker:latest`) on `AGY_PORT` (default 4400).
  Runs non-root (`developer` via `gosu`) with dynamic `PUID`/`PGID`, disabled
  passwordless sudo, and `umask 0002` for `conversations/`, `brain/`, and
  `annotations/`.
- **Host Execution:** Avoids `/var/run/docker.sock` exposure via an isolated
  SSH-based web terminal (`ttyd`) for host command execution.

## Configuration & Runtime Environment
- **Environment Variables:**
  - Auth & Host: `RC_NAME`, `AUTH_PASSWORD`, `HOST_SSH_DIR`.
  - Feature Flags: `ENABLE_IDE`, `ENABLE_TERMINAL` (both default `true`).
  - Networking: `AGY_PORT` (4400) and `AGY_HUB_PORT` (default 4402, passed via
    `--hub-port` to `agy --remote-control` for deterministic connections without
    scraping logs).
  - Telemetry: `BLOCK_TELEMETRY` (default `true`; sinkholes telemetry to
    `0.0.0.0` via `/etc/hosts` and sets OpenTelemetry opt-out variables).
- **Lifecycle & State:** Initial authentication via `setup` subcommand with
  mounted `~/.gemini`. Startup (`entrypoint.sh`) populates empty
  `$GEMINI_DIR/config/projects/` and purges stale candidate CSRF tokens. State
  (`antigravity_state.pbtxt` with `installation_uuid` and schema migrations)
  and logs (`cli.log`) reside in `$GEMINI_DIR/antigravity-cli/`.
- **Default Settings:** Sets `enableTerminalSandbox: true`,
  `autoExecutionPolicy: CASCADE_COMMANDS_AUTO_EXECUTION_PROCEED_IN_SANDBOX`,
  `nonWorkspaceFiles: ALLOW`, with sidebar shortcuts for VS Code IDE and Host
  Terminal.

## Auth Proxy & Gateway (`proxy/auth-proxy.js`)
- **Security & Middleware:** Dynamic 256-bit session tokens, in-memory session
  cleanup, IP rate-limiting on `/__auth/login`, 16 KB body limit, path-traversal
  guards, security headers (CSP, `X-Content-Type-Options`, `X-Frame-Options`),
  and centralized body parsing (`proxy/lib/security.js`).
- **Protocol Handling:** Enforces `useWebSocket=true` on `/` and `/c/...`, sets
  `X-Accel-Buffering: no`, flushes headers immediately, strips hop-by-hop
  headers, preserves `TE: trailers`, `Trailer`, and `grpc-status`, and prevents
  TCP RST packets during upstream socket teardown.
- **UI & Endpoints:** Unauthenticated `/status` health check (`200`/`503`),
  persistent favicon injection via `MutationObserver`, and cosmic glassmorphic
  UI with 2D canvas particles (`renderPageLayout`, `BASE_PAGE_CSS`).

## Sidecar Management (`proxy/sidecar-manager.js`)
- **Supervision & Lifecycle:** Supervises background workers and 5-field cron
  tasks via authenticated `/sidecars` REST APIs and UI. Supports restart
  policies, unified env resolution (`buildSidecarEnv`), upstream Language Server
  polling (`waitForUpstream()`), and live CSRF token querying at
  `127.0.0.1:${AGY_HUB_PORT}`.
- **Discovery & Types:**
  - Standalone: Defined in `~/.gemini/config/sidecars/<id>/sidecar.json` and
    toggled via `sidecars[id].enabled` in `~/.gemini/config/config.json`.
  - Plugin: Discovered at `<plugin>/sidecars/<name>/sidecar.json`, namespaced as
    `<plugin-name>/<sidecar-name>`, running with isolated `cwd`, prepended
    `PATH`, `PLUGIN` badges, and configuration resets.

## Testing & Quality
- **Test Runner & Isolation:** Native Node.js test runner (`node --test
  tests/*.js`). State isolation in `tests/test-sidecar-manager.js` restores mock
  environment variables and filesystem fixtures in `finally` blocks to prevent
  CSRF token pollution across test suites.

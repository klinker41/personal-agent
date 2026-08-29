---
topic: antigravity-docker
category: project
tags: [project, antigravity-docker]
updated_at: 2026-08-29T11:59:02Z
confidence: 0.95
---

# Project: Antigravity-Docker

## Overview & Deployment
- Published to Docker Hub at `jklinker/antigravity-docker:latest`. Docker
  Compose configurations and helper scripts pull this image directly.
- Provides a headless containerized environment for Google Antigravity (`agy`),
  exposing only port 4400 externally for direct or reverse-proxied access.
- Non-root `developer` execution is enforced via `gosu` for all services
  (proxy, `code-server`, `ttyd`, sidecars); passwordless sudo is disabled.
- Supports dynamic `PUID`/`PGID` and `umask 0002` to maintain proper
  permissions across `conversations/`, `brain/`, and `annotations/`.
- Replaced direct `/var/run/docker.sock` mounting with an SSH-based web terminal
  gateway (`ttyd`) to execute host commands securely without socket exposure.
- Keeps documentation and configurations generic across host environments
  rather than referencing specific platforms like Unraid.

## Container Configuration & Environment
- Consolidated compose configuration uses environment variables: `RC_NAME`,
  `AGY_PORT` (default 4400), `AUTH_PASSWORD`, `ENABLE_IDE` (default true),
  `ENABLE_TERMINAL` (default true), `HOST_SSH_DIR`, and `BLOCK_TELEMETRY`.
- `BLOCK_TELEMETRY` (default true) sinkholes Google telemetry endpoints to
  `0.0.0.0` via `/etc/hosts` and sets OpenTelemetry opt-out environment
  variables.
- Initial authentication is executed inside the container using the `setup`
  subcommand with `~/.gemini` mounted.
- Persistent state (`antigravity_state.pbtxt` containing `installation_uuid`
  and schema migrations) and CLI logs are stored strictly under
  `$GEMINI_DIR/antigravity-cli/` (checked by proxy at `.../cli.log`).
- `entrypoint.sh` initializes default project configs in
  `$GEMINI_DIR/config/projects` if empty; individual project configs are saved
  directly into the projects directory rather than a monolithic
  `projects.json`.
- Default Antigravity settings: `enableTerminalSandbox: true`,
  `autoExecutionPolicy: CASCADE_COMMANDS_AUTO_EXECUTION_PROCEED_IN_SANDBOX`,
  `nonWorkspaceFiles: ALLOW`, with VS Code IDE and Host Terminal sidebar
  buttons.

## Authentication Proxy & Gateway (`proxy/auth-proxy.js`)
- Protects endpoints with dynamic 256-bit session tokens, in-memory session
  cleanup, IP rate-limiting on `/__auth/login`, and a 16 KB body limit.
- Centralizes body accumulation (`readRequestBody`) and JSON parsing
  (`readJsonBody`) with byte limits and error handling in
  `proxy/lib/security.js`.
- Enforces security headers (CSP, `X-Content-Type-Options`, `X-Frame-Options`)
  and path traversal prevention on sidecar IDs.
- Serves unauthenticated `/status` health check (HTML or JSON `200`/`503`),
  direct favicon assets, and injects persistent favicons via `MutationObserver`.
- Custom glassmorphic cosmic UI with interactive 2D canvas particle simulation
  is shared across gateway pages via `renderPageLayout` and `BASE_PAGE_CSS`.
- Protocol & Stream Handling: Enforces `useWebSocket=true` on root and `/c/...`
  routes; sets `X-Accel-Buffering: no`; preserves `TE: trailers`, `Trailer`, and
  `grpc-status`; calls `res.flushHeaders()` immediately; strips hop-by-hop
  `Transfer-Encoding` headers; and cleans up upstream sockets without sending
  erroneous TCP RST packets via unconditional `proxyReq.destroy()`.

## Sidecar Management (`proxy/sidecar-manager.js`)
- Supervises background workers and scheduled prompts with 5-field cron
  parsing, restart policies, and unified environment resolution
  (`buildSidecarEnv`).
- Standalone sidecars: Definitions stored in
  `~/.gemini/config/sidecars/<id>/sidecar.json`, with enabled state in
  `~/.gemini/config/config.json` under `sidecars[id].enabled`.
- Plugin sidecars: Discovered from `<plugin>/sidecars/<name>/sidecar.json`,
  namespaced as `<plugin-name>/<sidecar-name>`, executed with `cwd` set and
  `PATH` prepended to sidecar folder, rendered with `PLUGIN` UI badge and
  configuration reset support.
- Sidecar Manager UI (`/sidecars`) and REST APIs are gated behind gateway auth.

## Testing
- Automated test suite runs via Node.js native test runner
  (`node --test tests/*.js`).

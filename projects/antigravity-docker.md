---
topic: antigravity-docker
category: project
tags: [project, antigravity-docker]
updated_at: 2026-09-22T00:32:07.015076+00:00
confidence: 0.95
---

# Project: Antigravity-Docker

## Container Runtime & Isolation
- **Security & Execution:** Runs headless `jklinker/antigravity-docker:latest`
  as non-root `developer` via `gosu` (dynamic `PUID`/`PGID`, disabled
  passwordless sudo, `umask 0002` across `conversations/`, `brain/`, and
  `annotations/`). Employs an isolated SSH web terminal gateway (`ttyd`)
  rather than mounting `/var/run/docker.sock`.

## Configuration & Environment
- **Networking & Ports:** Exposes `AGY_PORT=4400` (web interface/auth proxy) and
  `AGY_HUB_PORT=4402` (passed to `agy --remote-control --hub-port` for
  deterministic discovery).
- **Environment Flags:** Supports `RC_NAME`, `AUTH_PASSWORD`, and `HOST_SSH_DIR`
  for auth and host access; `ENABLE_IDE` and `ENABLE_TERMINAL` (default: `true`)
  for web tools; and `BLOCK_TELEMETRY=true` (sinkholes telemetry to `0.0.0.0`
  via `/etc/hosts` and sets OpenTelemetry opt-out variables).
- **Storage & Lifecycle:** Subcommand `setup` handles initial auth with mounted
  `~/.gemini`. `entrypoint.sh` initializes `$GEMINI_DIR/config/projects/` and
  purges stale CSRF tokens. Persistent state (`antigravity_state.pbtxt`,
  `installation_uuid`, migrations) and `cli.log` reside in
  `$GEMINI_DIR/antigravity-cli/`.
- **Default Policies & UI:** Enforces `enableTerminalSandbox: true`,
  `nonWorkspaceFiles: ALLOW`, and `autoExecutionPolicy:
  CASCADE_COMMANDS_AUTO_EXECUTION_PROCEED_IN_SANDBOX`, with sidebar navigation
  shortcuts for VS Code IDE and Host Terminal.

## Auth Proxy & Gateway (`proxy/auth-proxy.js`)
- **Security & Hardening:** Enforces 256-bit session tokens, in-memory session
  cleanup, IP rate-limiting on `/__auth/login`, 16 KB body limit, path
  traversal protection, security headers (CSP, frame/content-type options), and
  centralized body parsing in `proxy/lib/security.js`. Serves unauthenticated
  `/status` health checks.
- **Reverse Proxy 
<truncated 306 bytes>
viders:** Managed via `/models` UI
  (`proxy/lib/models-manager.js`), persisting configurations with masked API
  keys to `~/.gemini/config/custom_models.json`.

## Translation Proxy & Transcoding
- **Activation & Routing (`proxy/translation-proxy.js`):** Native Node.js
  streaming transcoder conditionally enabled by `entrypoint.sh` only when
  custom models are configured. Unregistered placeholder models in `M500`-`M649`
  return `null` immediately, routing built-in models (Claude, GPT-OSS)
  directly upstream to Google when Astra is active.
- **Argument Transcoding (`proxy/lib/transcoder.js`):** `sanitizeToolCallArgs`
  normalizes tool arguments across unary and streaming calls, coercing
  stringified booleans and integers into native types. Strips `ArtifactMetadata`
  outside `.gemini/antigravity-cli/brain/` (synthesizing `UserFacing: false`)
  and defaults `Overwrite: true` on non-artifact `write_to_file` calls when
  omitted by third-party models.

## Sidecar Management (`proxy/sidecar-manager.js`)
- **Supervisor Engine:** Authenticated `/sidecars` REST API and UI manages
  background workers and 5-field cron tasks. Coordinates restart policies,
  unified env injection (`buildSidecarEnv`), upstream Language Server polling
  (`waitForUpstream()`), and live CSRF token discovery at
  `127.0.0.1:${AGY_HUB_PORT}`.
- **Sidecar Types:**
  - *Standalone:* Defined in `~/.gemini/config/sidecars/<id>/sidecar.json` and
    toggled via `sidecars[id].enabled` in `~/.gemini/config/config.json`.
  - *Plugin:* Defined in `<plugin>/sidecars/<name>/sidecar.json`, namespaced as
    `<plugin-name>/<sidecar-name>`, executed with isolated `cwd`, prepended
    `PATH`, `PLUGIN` UI badge, and isolated config resets.

## Testing & Quality
- **Test Suite & Isolation:** Native Node test runner
  (`node --test tests/*.js`). Maintains state isolation by cleaning up mock
  environment variables and filesystem fixtures in `finally` blocks (e.g.,
  `tests/test-sidecar-manager.js`) to prevent CSRF token or state leakage
  between test suites.

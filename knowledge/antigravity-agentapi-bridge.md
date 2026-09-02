---
topic: antigravity-agentapi-bridge
category: knowledge
tags: [knowledge, antigravity-agentapi-bridge]
updated_at: 2026-09-02T00:00:39.988526+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Agentapi-Bridge

- Running or calling `agentapi new-conversation` with `ANTIGRAVITY_LS_ADDRESS`
  and `ANTIGRAVITY_CSRF_TOKEN` configured requires a valid, non-empty
  `PROJECT_ID` / `ANTIGRAVITY_PROJECT_ID` or it fails with 'project_id is
  required when providing project_env_config'.
- Daemon background tasks and subprocess bridges communicating with the
  Antigravity Language Server must clear ambient conversation metadata
  (`ANTIGRAVITY_SOURCE_METADATA`, `ANTIGRAVITY_CONVERSATION_ID`,
  `ANTIGRAVITY_TRAJECTORY_ID`) to avoid cross-project context errors and source
  project mismatch conflicts when targeting other workspaces.
- Language Server port and CSRF token change on server restarts; checking socket
  liveness and reading active values from `ls_address` and `cli.log` prevents
  stale connection failures.

- AgentApiBridge handles invalid or expired CSRF tokens by catching 'Unauthenticated (invalid CSRF token)' errors, fetching the active token from http://127.0.0.1:4402/, updating the runtime environment, and automatically retrying the command.

- The Antigravity Language Server exposes the active CSRF token via window.__APP_CONFIG__.csrfToken on its root HTTP hub endpoint (http://127.0.0.1:<PORT>/), allowing clients to dynamically fetch or refresh tokens when disk cache is stale.

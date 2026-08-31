---
topic: antigravity-agentapi-bridge
category: knowledge
tags: [knowledge, antigravity-agentapi-bridge]
updated_at: 2026-08-31T00:01:12.444063+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Agentapi-Bridge

- Calls to 'agentapi new-conversation' with ANTIGRAVITY_LS_ADDRESS and
ANTIGRAVITY_CSRF_TOKEN fail with 'project_id is required when providing
project_env_config' if PROJECT_ID/ANTIGRAVITY_PROJECT_ID is empty or missing
from the subprocess environment.
- Daemon background tasks invoking agentapi should strip caller metadata
(ANTIGRAVITY_SOURCE_METADATA, ANTIGRAVITY_CONVERSATION_ID,
ANTIGRAVITY_TRAJECTORY_ID) to avoid source project mismatch conflicts when
targeting other workspaces.
- Language Server port and CSRF token change on server restarts; checking socket
liveness and reading active values from ls_address and cli.log prevents stale
connection failures.

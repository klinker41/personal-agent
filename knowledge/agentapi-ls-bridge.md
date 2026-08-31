---
topic: agentapi-ls-bridge
category: knowledge
tags: [knowledge, agentapi-ls-bridge]
updated_at: 2026-08-31T00:00:46.580123+00:00
confidence: 0.95
---

# Knowledge: Agentapi-Ls-Bridge

- Running `agentapi new-conversation` with ANTIGRAVITY_LS_ADDRESS and
ANTIGRAVITY_CSRF_TOKEN configured requires a valid, non-empty PROJECT_ID or it
fails with 'project_id is required when providing project_env_config'.
- Subprocess bridges communicating with the Antigravity Language Server must
clear ambient conversation metadata (ANTIGRAVITY_SOURCE_METADATA,
ANTIGRAVITY_CONVERSATION_ID, ANTIGRAVITY_TRAJECTORY_ID) to avoid cross-project
context errors.

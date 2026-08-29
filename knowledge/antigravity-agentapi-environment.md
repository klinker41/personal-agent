---
topic: antigravity-agentapi-environment
category: knowledge
tags: [knowledge, antigravity-agentapi-environment]
updated_at: 2026-08-29T11:55:23.099751+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Agentapi-Environment

- Invoking agentapi from container daemon processes outside interactive chat
sessions requires injecting ANTIGRAVITY_LS_ADDRESS (e.g. 127.0.0.1:<PORT>) and
ANTIGRAVITY_CSRF_TOKEN into the execution environment.
- Missing ANTIGRAVITY_CSRF_TOKEN causes gRPC authentication errors ('missing
CSRF token') when attempting agentapi operations.
- The language server CSRF token can be dynamically discovered by fetching the root HTML from http://127.0.0.1:<PORT>/ and extracting window.__APP_CONFIG__.csrfToken.

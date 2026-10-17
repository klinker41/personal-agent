---
topic: agy-agentapi-subcommand
category: knowledge
tags: [knowledge, agy-agentapi-subcommand]
updated_at: 2026-10-17T00:01:17.103494+00:00
confidence: 0.95
---

# Knowledge: Agy-Agentapi-Subcommand

- When ANTIGRAVITY_AGENTAPI_EXE points to the agy binary
(/home/developer/.local/bin/agy), invocations must include the agentapi
subcommand (e.g., agy agentapi new-conversation) rather than invoking
subcommands directly on agy.

- `agy agentapi new-conversation` strictly accepts abstract tier keywords
(`flash_lite`, `flash` , `pro` ) via `--model` ; passing explicit model IDs like
`argon` or `gemini-4-argon-medium` fails with
`no available models found for tier <name>` .
- Upstream Google `TieredModelConfig` maps the `pro` tier to
`gemini-3.8-flash-tiered` (`MODEL_PLACEHOLDER_M322`) rather than Gemini 4 Argon
(`MODEL_PLACEHOLDER_M417` / `MODEL_PLACEHOLDER_M418` ), which are marked with a
'New' tag instead of the `pro` tier alias.

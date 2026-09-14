---
topic: antigravity-subagent-models
category: knowledge
tags: [knowledge, antigravity-subagent-models]
updated_at: 2026-09-14T00:04:01.948461+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Subagent-Models

- Antigravity's `invoke_subagent` tool supports specifying Gemini model tiers
(`pro`, `flash`, `flash_lite`, `inherit`) via the `Model` parameter.

- To avoid orchestrator agents over-delegating tasks, subagent rules should be
conditioned on subagent invocation (e.g., using Model: "flash" for coding or
testing) rather than mandating delegation.

---
topic: antigravity-pre-invocation-hooks
category: knowledge
tags: [knowledge, antigravity-pre-invocation-hooks]
updated_at: 2026-08-31T00:00:37.708165+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Pre-Invocation-Hooks

- In Antigravity PreInvocation hooks, checking `initialNumSteps == 0` alongside
  `invocationNum == 1` and inspecting transcript history ensures context is
  injected only on the initial conversation prompt, preventing redundant
  context payloads on subsequent turns.

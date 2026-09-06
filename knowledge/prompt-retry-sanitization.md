---
topic: prompt-retry-sanitization
category: knowledge
tags: [knowledge, prompt-retry-sanitization]
updated_at: 2026-09-06T00:01:23.716718+00:00
confidence: 0.95
---

# Knowledge: Prompt-Retry-Sanitization

- A reactive prompt sanitization strategy allows raw natural prompts to execute
first without preprocessing, invoking an LLM compliance rewriter only upon
safety or trademark rejection before retrying generation.

- When pipeline steps fail due to non-retryable `PROHIBITED_CONTENT` blocks,
implement automatic recovery using LLM-guided rewriting to replace trademarked
franchise lore and de-escalate violent action with generic equivalents before
retrying.

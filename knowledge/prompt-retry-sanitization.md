---
topic: prompt-retry-sanitization
category: knowledge
tags: [knowledge, prompt-retry-sanitization]
updated_at: 2026-09-15T00:00:32.249224+00:00
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

- Diagnostic reprompting loops achieve reliable error recovery by returning
structured validation diagnostics (syntax errors, schema boundary failures)
alongside sanitized response snippets rather than raw unparsed payloads.

- Preserving per-attempt history, raw response text, and intermediate diagnostic
reprompts across all retry cycles—on both success and exhaustion errors—enables
downstream LLMs to analyze and diagnose validation failure patterns during
post-mortems.

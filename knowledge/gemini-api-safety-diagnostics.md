---
topic: gemini-api-safety-diagnostics
category: knowledge
tags: [knowledge, gemini-api-safety-diagnostics]
updated_at: 2026-08-29T11:54:05.766056+00:00
confidence: 0.95
---

# Knowledge: Gemini-Api-Safety-Diagnostics

- When Gemini blocks output due to safety or policy filters (e.g., SAFETY,
BLOCKLIST, PROHIBITED_CONTENT), accessing response.text can return an empty
string without throwing a direct API error.
- To diagnose empty model outputs, inspect candidate finishReason,
safetyRatings, and promptFeedback.blockReason rather than assuming a successful
text generation.

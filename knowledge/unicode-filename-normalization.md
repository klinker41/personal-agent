---
topic: unicode-filename-normalization
category: knowledge
tags: [knowledge, unicode-filename-normalization]
updated_at: 2026-09-06T00:02:02.626604+00:00
confidence: 0.95
---

# Knowledge: Unicode-Filename-Normalization

- ASCII-only sanitization regexes like `/[a-zA-Z0-9]/` strip accented letters
and non-Latin scripts in filenames; use Unicode property classes
`[\p{L}\p{N}\p{M}]` to preserve letters, numbers, and combining marks.
- Always normalize filenames and user input to Unicode Normalization Form C
(`NFC`) when matching across platforms, as macOS and external mounts frequently
write filenames in decomposed form (`NFD`).
- When generating dynamic RegExp patterns from sanitized candidate strings for
fuzzy lookup, guard against inputs consisting entirely of stripped characters to
avoid empty regexes that falsely match all strings.

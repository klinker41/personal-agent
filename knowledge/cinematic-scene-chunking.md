---
topic: cinematic-scene-chunking
category: knowledge
tags: [knowledge, cinematic-scene-chunking]
updated_at: 2026-09-08T00:01:25.155331+00:00
confidence: 0.95
---

# Knowledge: Cinematic-Scene-Chunking

- LLMs planning video scene chunks tend to default entirely to camera cuts when
shot/reverse-shot coverage rules are enforced; they require explicit heuristics
for continuous camera takes (e.g., dialogue continuation, tracking movement,
sustained shared staging) to maintain continuity across sequential chunks.

- Avoid explicit trigger words (e.g., 'blood' and graphic gore references) in
scene chunking system instructions to prevent tripping gateway safety filters
while retaining dramatic PG-13 action pacing.

- Dialogue chunking engines favor camera cuts (`new_shot`) over continuous shots
because shot/reverse-shot rules trigger a new camera setup on character
alternations.
- Continuous shots (`camera_continuity: 'continuous'`) are primarily reserved
for dialogue continuation when a single character's speech exceeds per-chunk
length limits (e.g., 10 seconds or 18–20 words).

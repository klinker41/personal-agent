---
topic: procedural-cel-rigging
category: knowledge
tags: [knowledge, procedural-cel-rigging]
updated_at: 2026-11-15T07:01:13.130021+00:00
confidence: 0.95
---

# Knowledge: Procedural-Cel-Rigging

- Prevent detached segments and pinched joints in procedural 2D cel character
rigs by rendering limbs with concentric two-pass tapered capsules and validating
rendered poses via pixel flood-fill after `1px` erosion to catch multi-component
gaps or `<3px` bridges.

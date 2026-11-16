---
topic: animation-generator
category: project
tags: [project, animation-generator]
updated_at: 2026-11-15T17:44:47.061650+00:00
confidence: 0.95
---

# Project: Animation-Generator

- Projects under `projects/<slug>/` organize multi-scene adaptations with
`chapters/`, `voices.json`, `soundtrack.json`, `assets/characters/`, per-scene
`scenes/<slug>/` (`script.json`, `scene.ts`, `qa_sheet.png`, `scene.mkv`), and a
stitched `output/movie.mkv` master with stem-mixed audio and embedded `.srt`
subtitles.
- The `.agents/skills/cel-animation/` workspace skill provides concentric
two-pass limb rigging primitives (`renderTaperedCapsulePath`, `renderCelArm`,
`renderCelLeg`, `renderCelNeck`, `renderCharacterModelSheet`) in
`scripts/rigging.ts` (re-exported via `scripts/common.ts`).
- `scripts/render_asset.ts` runs `auditPoseConnectivity` (pixel flood-fill plus
`1px` erosion) to enforce single-component character silhouettes and reject
detached limbs, floating hands/heads, or `<3px` pinched joints across all poses.

- Character and scene assets are generated as programmatic TypeScript SVG
modules rasterized via `@resvg/resvg-js` (bundling `Outfit` and `Cinzel` fonts)
in `.agents/skills/cel-animation/scripts/` (`render_asset.ts`,
`render_scene.ts`, `common.ts`) rather than calling an image generation model.
- The cel-animation visual aesthetic (bold ink outlines, flat fills, stepped cel
shadows, two-pass `<rect>` capsule limbs, and standardized model-sheet layout
with `front`/`three_quarter`/`side` views, 5 visemes, and blink states) is
enforced via workspace and skill instructions in `AGENTS.md` and
`.agents/skills/cel-animation/SKILL.md`.

- Stored on unraid-nas at /mnt/user/ai/animation-generator and bind-mounted into
the antigravity container (stacks/unraid-nas/ai/compose.yaml) at
/workspace/animation-generator.

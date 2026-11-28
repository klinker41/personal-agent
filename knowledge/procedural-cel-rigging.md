---
topic: procedural-cel-rigging
category: knowledge
tags: [knowledge, procedural-cel-rigging]
updated_at: 2026-11-28T07:03:46.736951+00:00
confidence: 0.95
---

# Knowledge: Procedural-Cel-Rigging

- Prevent detached segments and pinched joints in procedural 2D cel character
rigs by rendering limbs with concentric two-pass tapered capsules and validating
rendered poses via pixel flood-fill after `1px` erosion to catch multi-component
gaps or `<3px` bridges.

- To prevent moonwalking and sliding illusions in 2D rigged walks, drive stride
phase from signed horizontal displacement (`worldX`) relative to computed step
span (`computeStepSpanPx`), use full side-profile (`angle: 'side'`) rigs rather
than `three_quarter` rigs for lateral traversal, and export per-frame locomotion
telemetry (`worldX`, `facing`, `walkPhase`, `walkAmount`, `stepSpanPx`) to
programmatically detect backward motion, rigid sliding, phase snaps, and
walk-in-place.
- For seamless barefoot legs in two-pass cel-shaded SVG rigs, start the barefoot
path with an ankle-pivot semicircle `(-rAnkle, 0) -> (rAnkle, 0)` so ankle
rotation stays flush with the shin across every stance angle, and merge barefoot
geometry into both Pass 1 (unified outer ink silhouette across thigh, shin, and
foot) and Pass 2 (stroke-free skin fill with sole shadow and toe contour) to
eliminate horizontal ankle seam lines.

- Straight tapered-capsule limbs (renderTaperedCapsulePath) and flat
side-scroller horizons produce a blocky 2D puppet look regardless of palette or
outline weight; replacing straight tubes with sculpted cubic-Bézier limb paths
(renderSculptedLimbPath) while retaining circular joint pivots preserves
auditPoseConnectivity compatibility and pairs with 2-point perspective staging
to break the flat side-scroller look within hand-coded SVG.
- computeStridePhase takes a numeric stepSpanPx argument rather than an options
object; passing an object yields NaN in walkPhase and invalid SVG transforms
like rotate(NaN) that detach rigged parts (e.g. character heads), so sampled SVG
frames should be audited for NaN and undefined strings.
- Letterbox bars and HUD overlays must be rendered in screen-locked coordinates
rather than world space to prevent camera transforms from causing world-space
letterbox bleed.

- To prevent procedurally rigged SVG hands from looking detached or bolted onto
arms, terminate forearm paths flush at the wrist (with an anatomical taper for
bare forearms) rather than using protruding semicircular endcaps that poke past
cuffs or bulge around wrists.
- Avoid closed standalone wrist-stem polygons on rigged SVG hands, which draw a
hard horizontal ink cut-line across the wrist; instead, use an open-top Pass 1
outer contour (`stroke-linecap="butt"` aligned flush to the forearm edges)
paired with a stroke-free Pass 2 interior fill tucked into the wrist or cuff.
- Couple hand dimensions directly to forearm wrist width (`wWrist`, `palmScale`)
and mirror thumb orientation (`thumbDir`) for front-view right arms to keep
hands anatomically seamless across poses.

- Programmatic silhouette connectivity checks (confirming a rig renders as a
single connected component) cannot catch anatomical or perspective defects such
as missing necks (head touching torso), backward-bending elbows/knees, swapped
near/far arms in 3/4 view, or sideways feet in front view; visual model-sheet
auditing is still required.
- Delegating visual and audio QA of rendered animation assets to a fresh
read-only subagent (`research` on `flash`) provided only with rendered outputs
and script/soundtrack specs mitigates authoring confirmation bias compared to
self-reviewing in the authoring turn.

---
topic: compose-focus-navigation
category: knowledge
tags: [knowledge, compose-focus-navigation]
updated_at: 2026-11-27T07:03:10.292923+00:00
confidence: 0.95
---

# Knowledge: Compose-Focus-Navigation

- When pairing a custom focus ring modifier (such as `focusOutline`) with
`Modifier.clickable`, pass an explicit `ripple()` `indication` so Compose's
default indication does not render a duplicate focus ring.
- Guard initial `FocusRequester.requestFocus()` calls inside scrollable headers
(such as a TV hero carousel) with a `rememberSaveable` boolean flag so scrolling
back to the top and recomposing does not re-capture focus.
- Chain `.focusGroup()` after `.focusRestorer()` on Compose navigation drawer
containers to scope focus restoration properly.

- Custom `LocalIndication` providers (such as D-pad/TV focus rings) must be
placed inside `MaterialTheme`'s content lambda rather than wrapping
`MaterialTheme`, because `MaterialTheme` unconditionally provides its own ripple
`LocalIndication` and overrides outer providers.
- For D-pad/remote video player controls where the HUD auto-hides, track
`isHoldRepeat` on consumed `DPAD_LEFT`/`DPAD_RIGHT` key-down events so holding
to seek from a hidden HUD continues seeking across key repeats rather than
switching mid-hold to focus traversal; pass `Ctrl`, `Left Alt`, and `Meta`
modifier chords through to `super.dispatchKeyEvent` before custom remote
handling.

- When attaching an initial FocusRequester to a Compose row that uses
focusRestorer(), place focusRequester() immediately before focusRestorer() in
the modifier chain.
- Default bring-into-view behavior inside a LazyColumn only scrolls far enough
to reveal a focused child button inside a top hero banner; explicitly scroll the
LazyColumn to (0, 0) on hero focus so the full banner image is visible.
- When replacing focus outlines with a custom scale-up FocusIndication in
Compose, use TraversableNode to prevent nested focusable components from
compounding scale transforms, and adjust modifier clip ordering and card spacing
so scaled items are not clipped.

- In Jetpack Compose, opening a dialog from
`combinedClickable(onLongClick = ...)` while a D-pad or keyboard activation key
(`DirectionCenter`, `Enter`, `NumPadEnter`, `Spacebar`) is still held down
causes `AbstractClickableNode` on the newly focused dialog button to accept
repeated `KeyDown` events (`repeatCount > 0`) and immediately fire `onClick` on
`KeyUp` when the user releases the key.
- To prevent a held long-press key from accidentally clicking the initially
focused button in a newly opened Compose dialog, attach
`Modifier.onPreviewKeyEvent` to the dialog to consume repeated activation
`KeyDown` events (`repeatCount > 0`), track fresh `KeyDown` (`repeatCount == 0`)
events per key, and consume unmatched `KeyUp` events.

- When implementing custom D-pad Left/Right scrubbing on a Material 3 `Slider`,
consume both `KeyDown` and `KeyUp` events for `DirectionLeft` and
`DirectionRight` so Material 3's internal `Slider` `KeyUp` handler does not
trigger unintended step jumps.
- TV modal overlays in Compose require an unfocusable scrim backdrop, focus
trapping inside the panel, explicit initial focus assignment, `Back`/`Escape`
dismissal handling, and focus restoration to the triggering control on close.

- Under Compose TV's `PivotBringIntoViewSpec` (30% viewport pivot), focusing
sibling buttons inside a top hero carousel can scroll the parent list partially
down; attaching a `BringIntoViewModifierNode` to the hero container that pins
child `BringIntoView` rects to `(0, 0)` keeps the carousel anchored at the top.
- Replacing layout-layer focus scaling with draw-phase focus overlays (border
halo and gradient sheen) avoids vertical `BringIntoView` scroll bounce during
horizontal focus traversal across row items; hoist static `Shape` instances
outside draw modifiers to prevent per-frame allocations.
- Robolectric tests verifying TV scroll positioning must configure the scroll
container with a `BringIntoViewSpec` matching `PivotBringIntoViewSpec` to
reproduce Compose TV's 30% pivot behavior.

- Material 3's `MinimumInteractiveModifierNode` expands `36.dp`/`40.dp`
`IconButton`s to `48.dp`, causing oversized focus highlights that clip against
`TopAppBar`'s `clipToBounds()`; provide `LocalMinimumInteractiveComponentSize
provides Dp.Unspecified` on TV where touch-target padding is unneeded.
- To avoid clipping scaled focus rings (`focusOutline`) on pill-grouped or
badge-cutout buttons, apply pill shapes via `.background(color, shape)` rather
than `.clip(shape)` on the parent row, and place focus/scale/clickable modifiers
before `.graphicsLayer { compositingStrategy = CompositingStrategy.Offscreen }`
on the same node.
- When coordinating a parent fallback `FocusRequester` with a child item's
single-frame deferred focus retry on screen return, await two frames
(`withFrameNanos {}` twice) before invoking the parent fallback so child focus
restoration settles first.

- Touch-oriented floating overlays such as scroll-to-top `FloatingActionButton`s
inside `AnimatedVisibility` join the Compose focus tree once revealed and can
hijack D-pad directional focus from `LazyColumn` rows on Android TV; gating
`AnimatedVisibility` visibility on `!isTelevision` omits the node from
composition and the focus hierarchy completely.

- Explicitly setting `.focusProperties { canFocus = true }` overrides Compose's
`Focusability.SystemDefined` behavior (which keeps controls non-focusable in
`InputMode.Touch`), causing buttons to capture focus on touchscreens when
containers like `NavigableListDetailPaneScaffold` request pane focus; only
override `canFocus = false` when disabling focusability, leaving system-defined
focusability intact when enabled.
- Custom focus indication nodes (such as TV scale, border, halo, and sheen
effects) should gate rendering against `InputMode.Touch` on non-TV devices
rather than relying solely on focus state, ensuring programmatic pane focus does
not display TV highlights on touchscreens.

- When animating a Compose `Slider` thumb's visual size on TV or keyboard focus,
place the indicator inside a fixed-size thumb container matching its maximum
focused dimension so resizing the thumb does not shift the slider's track
bounds.

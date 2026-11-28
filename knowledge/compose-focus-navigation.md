---
topic: compose-focus-navigation
category: knowledge
tags: [knowledge, compose-focus-navigation]
updated_at: 2026-11-28T07:53:32.879344+00:00
compacted_at: 2026-11-28T07:53:32.879344+00:00
confidence: 0.95
---

# Knowledge: Compose Focus & Navigation

## Focus Indication, Scaling & Clipping

- **Custom `LocalIndication` placement**: Provide custom `LocalIndication`
  (e.g., D-pad/TV focus rings) inside `MaterialTheme`'s content lambda rather
  than wrapping `MaterialTheme`, which unconditionally overrides it with
  ripple. When pairing a custom focus ring modifier (such as `focusOutline`)
  with `Modifier.clickable`, pass an explicit `ripple()` `indication` so
  Compose's default indication does not render a duplicate ring.
- **Gate focus visuals on touch devices**: Custom focus indication nodes (TV
  scale, border, halo, sheen) should check against `InputMode.Touch` on non-TV
  devices rather than relying solely on focus state so programmatic pane focus
  does not show TV highlights on touchscreens.
- **Draw-phase vs. scale-up focus visuals**: Prefer draw-phase focus overlays
  (border halo and gradient sheen) over layout-layer scaling to avoid vertical
  `BringIntoView` scroll bounce during horizontal row traversal, and hoist
  static `Shape` instances outside draw modifiers to avoid per-frame
  allocations. If using a scale-up `FocusIndication`, use `TraversableNode` to
  prevent nested focusables from compounding scale transforms, and adjust card
  spacing and clip ordering so scaled items are not clipped.
- **Prevent clipping on buttons and bars**:
  - On TV (where touch-target padding is unneeded), provide
    `LocalMinimumInteractiveComponentSize provides Dp.Unspecified` so
    Material 3's `MinimumInteractiveModifierNode` does not expand
    `36.dp`/`40.dp` `IconButton`s to `48.dp` and clip oversized focus
    highlights against `TopAppBar`'s `clipToBounds()`.
  - For pill-grouped or badge-cutout buttons with scaled focus rings
    (`focusOutline`), apply pill shapes via `.background(color, shape)` rather
    than `.clip(shape)` on the parent row, and place focus, scale, and
    clickable modifiers on the same node before
    `.graphicsLayer { compositingStrategy = CompositingStrategy.Offscreen }`.

## Focusability, Restoration & Timing

- **Preserve `Focusability.SystemDefined`**: Do not explicitly set
  `.focusProperties { canFocus = true }`; it overrides
  `Focusability.SystemDefined` (which keeps controls non-focusable in
  `InputMode.Touch`) and causes buttons to capture focus on touchscreens when
  containers like `NavigableListDetailPaneScaffold` request pane focus. Only
  override `canFocus = false` when disabling focusability.
- **Keep touch-only overlays out of TV D-pad traversal**:
  - Gate touch-oriented floating overlays (e.g., scroll-to-top
    `FloatingActionButton`s inside `AnimatedVisibility`) on `!isTelevision` so
    they are omitted from composition and never hijack `LazyColumn` D-pad focus
    on Android TV.
  - Apply `focusProperties { canFocus = false }` to clickable labels or toggles
    placed directly above a video player seekbar (e.g., a remaining-time
    toggle) so touch users can still tap them without intercepting Android TV
    D-pad up-navigation between the seekbar and center playback controls.
- **`focusRestorer()` modifier ordering**: Place an initial `focusRequester()`
  immediately before `focusRestorer()` on rows, and chain `.focusGroup()` after
  `.focusRestorer()` on navigation drawer containers to scope restoration
  properly.
- **Initial and fallback focus timing**:
  - Guard initial `FocusRequester.requestFocus()` calls inside scrollable
    headers (such as a TV hero carousel) with a `rememberSaveable` boolean flag
    so scrolling back to the top and recomposing does not re-capture focus.
  - When coordinating a parent fallback `FocusRequester` with a child item's
    single-frame deferred focus retry on screen return, await two frames
    (`withFrameNanos {}` twice) before invoking the parent fallback so child
    focus restoration settles first.

## Scroll & BringIntoView on TV

- **Keep top hero banners anchored at `(0, 0)`**: Default `LazyColumn`
  bring-into-view only scrolls far enough to reveal a focused child button
  inside a top hero banner, and Compose TV's `PivotBringIntoViewSpec` (30%
  viewport pivot) can scroll the parent list partially down when focusing
  sibling buttons in a hero carousel. Explicitly scroll the `LazyColumn` to
  `(0, 0)` on hero focus so the full banner image is visible, and attach a
  `BringIntoViewModifierNode` to the hero container that pins child
  `BringIntoView` rects to `(0, 0)`.
- **Robolectric TV scroll tests**: Configure scroll containers with a
  `BringIntoViewSpec` matching `PivotBringIntoViewSpec` to reproduce Compose
  TV's 30% pivot behavior.

## Modals, Key Events & Media Controls

- **TV modal overlays**: Require an unfocusable scrim backdrop, focus trapping
  inside the panel, explicit initial focus assignment, `Back`/`Escape`
  dismissal handling, and focus restoration to the triggering control on close.
- **Prevent held long-press keys from clicking dialog buttons**: Opening a
  dialog from `combinedClickable(onLongClick = ...)` while a D-pad or keyboard
  activation key (`DirectionCenter`, `Enter`, `NumPadEnter`, `Spacebar`) is
  still held causes `AbstractClickableNode` on the newly focused dialog button
  to accept repeated `KeyDown` events (`repeatCount > 0`) and immediately fire
  `onClick` on `KeyUp`. Attach `Modifier.onPreviewKeyEvent` to the dialog to
  consume repeated activation `KeyDown` events (`repeatCount > 0`), track fresh
  `KeyDown` (`repeatCount == 0`) per key, and consume unmatched `KeyUp` events.
- **Auto-hiding video player HUD**: Track `isHoldRepeat` on consumed
  `DPAD_LEFT`/`DPAD_RIGHT` key-down events so holding to seek from a hidden HUD
  continues seeking across key repeats rather than switching mid-hold to focus
  traversal. Pass `Ctrl`, `Left Alt`, and `Meta` modifier chords through to
  `super.dispatchKeyEvent` before custom remote handling.
- **Material 3 `Slider` on TV/keyboard**:
  - For custom D-pad Left/Right scrubbing, consume both `KeyDown` and `KeyUp`
    events for `DirectionLeft` and `DirectionRight` so Material 3's internal
    `Slider` `KeyUp` handler does not trigger unintended step jumps.
  - When animating a `Slider` thumb's visual size on focus, place the indicator
    inside a fixed-size thumb container matching its maximum focused dimension
    so resizing the thumb does not shift the slider's track bounds.

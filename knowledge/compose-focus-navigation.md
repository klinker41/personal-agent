---
topic: compose-focus-navigation
category: knowledge
tags: [knowledge, compose-focus-navigation]
updated_at: 2026-11-25T07:14:19.659585+00:00
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

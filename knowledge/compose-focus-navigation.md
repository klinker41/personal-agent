---
topic: compose-focus-navigation
category: knowledge
tags: [knowledge, compose-focus-navigation]
updated_at: 2026-11-24T07:05:57.321753+00:00
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

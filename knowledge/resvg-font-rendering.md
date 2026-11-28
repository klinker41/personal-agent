---
topic: resvg-font-rendering
category: knowledge
tags: [knowledge, resvg-font-rendering]
updated_at: 2026-11-28T07:01:07.783115+00:00
confidence: 0.95
---

# Knowledge: Resvg-Font-Rendering

- `@resvg/resvg-js` defaults variable TTF fonts (`fvar` table present) to their
lightest weight axis instance (e.g., `wght=100` Thin/Hairline for `Outfit.ttf`
and `Cinzel.ttf`), rendering SVG text overly slim and difficult to read when
downscaled from 4K.
- Bundle static per-weight TTF font files (`fvar=False`, such as Regular 400,
SemiBold 600, Bold 700, ExtraBold 800) instead of variable fonts when rendering
crisp, bold typography via `@resvg/resvg-js`.

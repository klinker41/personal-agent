---
topic: trmnl-eink-rendering
category: knowledge
tags: [knowledge, trmnl, eink, pillow, webhook]
updated_at: 2026-10-30T14:36:10.703208+00:00
compacted_at: 2026-10-30T14:36:10.703208+00:00
confidence: 0.95
---

# Knowledge: TRMNL E-Ink & Webhook Plugin Rendering

- TRMNL e-ink display standard resolution is 800x480; direct Pillow rendering
with TrueType fonts and vector icons produces significantly sharper contrast
than downsampling headless browser screenshots.
- Generating 800x480 e-ink images directly via Pillow takes ~18ms and consumes
~30-50 MB RAM, avoiding the ~1 GB memory footprint of headless Chromium
automation.

- TRMNL does not execute client-side JavaScript; HTML markup is rendered
server-side without evaluating `<script>` tags or DOM events.
- TRMNL private plugins only re-evaluate Liquid/HTML templates on data events
(Polling URL updates or incoming Webhooks); without data triggers, TRMNL serves
a cached 800x480 bitmap without re-fetching static image URLs.
- The TRMNL 'Webhook Image' plugin accepts raw PNG byte payloads with
`Content-Type: image/png` to update display images directly without intermediate
hosting.

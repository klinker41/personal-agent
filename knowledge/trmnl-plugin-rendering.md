---
topic: trmnl-plugin-rendering
category: knowledge
tags: [knowledge, trmnl-plugin-rendering]
updated_at: 2026-08-29T11:55:28.214076+00:00
confidence: 0.95
---

# Knowledge: Trmnl-Plugin-Rendering

- TRMNL does not execute client-side JavaScript; HTML markup is rendered
server-side without evaluating `<script>` tags or DOM events.
- TRMNL private plugins only re-evaluate Liquid/HTML templates on data events
(Polling URL updates or incoming Webhooks); without data triggers, TRMNL serves
a cached 800x480 bitmap without re-fetching static image URLs.
- The TRMNL 'Webhook Image' plugin accepts raw PNG byte payloads with
`Content-Type: image/png` to update display images directly without intermediate
hosting.

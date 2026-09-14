---
topic: antigravity-container-setup
category: knowledge
tags: [knowledge, antigravity-container-setup]
updated_at: 2026-09-14T00:01:40.239071+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Container-Setup

- Interactive Antigravity CLI container authentication can be executed via:
docker run -it --rm -v '<host-path>:/home/developer/.gemini'
jklinker/antigravity-docker:latest setup

- Minimal container environments may lack `/usr/share/fonts/` and fontconfig
configuration, causing SVG `<text>` elements processed by librsvg to render as
tofu glyphs unless TrueType fonts are bundled directly.

- Without sudo access in the container, standalone static binaries such as
FFmpeg can be installed into /home/developer/.local/bin, which is already
present on PATH.

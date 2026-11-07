---
topic: antigravity-container-setup
category: knowledge
tags: [knowledge, antigravity-container-setup]
updated_at: 2026-11-07T07:03:08.125923+00:00
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

- Antigravity accesses the host Docker daemon and Komodo stacks and host volumes
without mounting `/var/run/docker.sock` by leveraging host environment variables
(`HOST_SSH_HOST`, `HOST_SSH_USER`, `HOST_SSH_PORT`) and an authorized SSH key
bind-mounted at `/home/developer/.ssh` from `/mnt/user/docker/antigravity/ssh`.

- Because `agy` is installed dynamically during `docker build` via
`https://antigravity.google/cli/install.sh`, the shipped CLI version can be
extracted post-build using `docker run --rm --entrypoint agy <image> --version |
tr -d '[:space:]'` to tag release images (`<AGY_VERSION>` and
`<AGY_VERSION>-slim`).

---
topic: gow-nvidia-driver
category: knowledge
tags: [knowledge, gow-nvidia-driver]
updated_at: 2026-11-22T07:03:50.189361+00:00
confidence: 0.95
---

# Knowledge: Gow-Nvidia-Driver

- After upgrading the Unraid host Nvidia driver, the server must reboot to load
the new kernel module before rebuilding gow/nvidia-driver:latest and recreating
nvidia-driver-vol so Games on Whales user-space libraries match the active host
driver.

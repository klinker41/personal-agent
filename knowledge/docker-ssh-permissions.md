---
topic: docker-ssh-permissions
category: knowledge
tags: [knowledge, docker-ssh-permissions]
updated_at: 2026-08-29T11:56:01.949048+00:00
confidence: 0.95
---

# Knowledge: Docker-Ssh-Permissions

- OpenSSH inside containers strictly enforces permissions (`0700` on `~/.ssh`,
  `0600` on private keys) and matching container UID ownership, silently
  failing key discovery if created by root or with broad permissions
  (`0644`/`0777`); mounted SSH keys should be staged to a container-local path
  with `chmod 0600`.
- Setting `StrictHostKeyChecking accept-new` in `~/.ssh/config` or
  pre-populating known host keys avoids interactive verification failures
  during non-interactive container SSH and Git operations.

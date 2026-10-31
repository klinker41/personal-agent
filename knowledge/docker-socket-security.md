---
topic: docker-socket-security
category: knowledge
tags: [knowledge, docker, ssh, security, git, containers]
updated_at: 2026-10-30T18:59:00+00:00
compacted_at: 2026-10-30T18:59:00+00:00
confidence: 0.95
---

# Knowledge: Docker Security, SSH Permissions & Git Safe Directories

- Mounting the host Docker socket (`/var/run/docker.sock`) inside containers
  running automated execution scripts introduces critical host takeover and
  container escape risks.
- Connecting to a Unix domain socket (`connect(2)`) inside a Docker container
  requires write permission on both the socket file and the bind mount; mounting
  a directory containing a Unix socket as read-only (`:ro`) causes client
  connections (such as `wolf-den` connecting to `/var/run/wolf/wolf.sock`) to
  fail even if the socket file exists.
- OpenSSH inside containers strictly enforces permissions (`0700` on `~/.ssh`,
  `0600` on private keys) and matching container UID ownership, silently failing
  key discovery if created by root or with broad permissions (`0644`/`0777`);
  mounted SSH keys should be staged to a container-local path with `chmod 0600`.
- Setting `StrictHostKeyChecking accept-new` in `~/.ssh/config` automatically
  records new host keys on first connection while protecting against changed
  keys, avoiding interactive verification failures during non-interactive
  container SSH and Git operations.
- When an exec-form JSON array in a Dockerfile `CMD` instruction is malformed or
  unclosed, Docker falls back to shell form (`/bin/sh -c '...'`), which causes
  runtime syntax errors such as
  `/bin/sh: 1: Syntax error: Unterminated quoted string` rather than a
  build-time failure.
- Wildcard patterns can be used in git `safe.directory` configuration to trust
  multiple repositories (e.g., `git config --global --add safe.directory
  '/workspace/*'`). If `git config --global` fails due to bind-mounted file
  locks, add the entry directly to `~/.gitconfig` under `[safe]`
  (`directory = /workspace/*`).

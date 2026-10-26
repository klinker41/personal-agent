---
topic: docker-socket-security
category: knowledge
tags: [knowledge, docker-socket-security]
updated_at: 2026-10-26T06:03:07.318061+00:00
confidence: 0.95
---

# Knowledge: Docker-Socket-Security

- Mounting the host Docker socket (/var/run/docker.sock) inside containers
running automated execution scripts introduces critical host takeover and
container escape risks.

- Connecting to a Unix domain socket (`connect(2)`) inside a Docker container
requires write permission on both the socket file and the bind mount; mounting a
directory containing a Unix socket as read-only (`:ro`) causes client
connections (such as `wolf-den` connecting to `/var/run/wolf/wolf.sock` ) to
fail even if the socket file exists.

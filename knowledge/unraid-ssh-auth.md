---
topic: unraid-ssh-auth
category: knowledge
tags: [knowledge, unraid-ssh-auth]
updated_at: 2026-08-29T11:57:01.235625+00:00
confidence: 0.95
---

# Knowledge: Unraid-Ssh-Auth

- On Unraid, `/boot/config/ssh/root/authorized_keys` resides on a FAT32
filesystem with 777 permissions, causing OpenSSH with `StrictModes yes` to
refuse key authentication with 'bad ownership or modes'.
- To allow SSH public key authentication on Unraid FAT32 flash configs, set
`StrictModes no` in `/etc/ssh/sshd_config`, reload via `/etc/rc.d/rc.sshd
reload`, and persist the command in `/boot/config/go`.

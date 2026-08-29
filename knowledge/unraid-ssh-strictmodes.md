---
topic: unraid-ssh-strictmodes
category: knowledge
tags: [knowledge, unraid-ssh-strictmodes]
updated_at: 2026-08-29T11:43:05.133633+00:00
confidence: 0.95
---

# Knowledge: Unraid-Ssh-Strictmodes

- Unraid flash storage (/boot) is formatted as FAT32 (vfat) with 777
permissions. OpenSSH sshd rejects authorized_keys files stored on /boot due to
StrictModes; set 'StrictModes no' in /etc/ssh/sshd_config to allow key-based SSH
authentication.

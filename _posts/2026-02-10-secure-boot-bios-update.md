---
layout: post
title:  "Fix secure boot after BIOS update"
date:   2026-02-10 07:36:00 +0200
categories: linux
---

1. Disable secure boot in firmware.
2. Boot into debian
3. dpkg-reconfigure shim-unsigned
4. Reboot
5. Enable secure boot in firmware.

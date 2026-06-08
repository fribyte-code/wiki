+++
title = "Firewall"
description = "Open ports in the firewall"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/instrukser/brannmur.md"
+++

## Open ports:

1. Go to andeby `ssh root@andeby.fribyte.no`
2. `ssh dole`
3. Edit `/etc/pf.conf`
4. Synchronize things:
5. `./syncpf.sh` -> `q` -> `y` -> Success

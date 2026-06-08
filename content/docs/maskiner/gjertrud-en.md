+++
title = "Gjertrud"
description = "Overview of Gjertrud"
template = "docs/page.html"
sort_by = "weight"
weight = 10001
draft = false

[extra]
lang = "en"
translation = "docs/maskiner/gjertrud.md"
+++

## Gjertrud

# Storage setup

Gjertrud is currently configured with all disks set up as their own RAID 1
through the standard RAID card built into the server motherboard. This should be
upgraded in the future to a Dell H200 (flashed to LSI 9211-8i IT mode).

All disks in this server are connected into a ZFS pool so that we can use
replication between Gjertrud and Konrad.

# Things learned

To be able to use an external HBA card in this server:

1. Disable the built-in RAID card
2. Upgrade the BIOS

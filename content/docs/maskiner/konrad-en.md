+++
title = "Konrad"
description = "Overview of Konrad"
template = "docs/page.html"
sort_by = "weight"
weight = 10001
draft = false

[extra]
lang = "en"
translation = "docs/maskiner/konrad.md"
+++

# Konrad

**NB!** Konrad has problems saving BIOS configuration. It is VERY IMPORTANT
that when Konrad is rebooted, you must choose the boot device manually. Go into
BIOS under boot, and select the boot option with "Linux" in the name.

## Proxmox

The server runs Proxmox VE 7.2 and is in the same cluster as Gjertrud. The web
panel can be reached over WireGuard through Gjertrud. Otherwise, it can be
reached via an SSH tunnel:

    ssh -L 8006:158.37.6.28:8006 root@bestemor.ss.uib.no

Then navigate to `localhost:8006` in the browser.

The plan is to run replication between Konrad and Gjertrud. TODO

## Storage setup

Konrad currently has a Dell H200 SATA HBA card (flashed to LSI 9211-8i IT
mode), connected to the SAS backplane in the server.

There are 4 1 TB SSDs connected to the HBA card, and 4 1 TB HDDs connected to
the motherboard. These are each placed in their own ZFS RAIDZ1 pool, named
`rpool` and `storpool` respectively. The Proxmox installation is on `rpool`.

## Documentation of how Konrad was set up

We set up the server with storage as described under
[storage setup](#storage-setup). Some of the disks had been configured in RAID
with LVM; these were wiped using `lvremove` and `mdadm`. We previously tried a
different Areca HBA card, which did not work well in this server.

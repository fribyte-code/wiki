+++
title = "Bolivar Skaftetryne Pluto"
description = "Description of setup on Bolivar Skaftetryne and Pluto"
date = 2023-05-02T16:19:00+00:00
updated = 2023-05-02T16:19:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 1
draft = false
+++

## Bolivar Skaftetryne Pluto

These servers work as a 3 node Proxmox cluster.

They are configured to share a ZFS pool, this allows us to use High Availability so that if any of the nodes go down, the VM's will be spun up on one of the other two. 


## Physical setups:

## HBA cards

The servers use RAID cards set to IT mode (essentially HBA cards). This is because ZFS does not support running with a raid card, as it wants individual control of the drives. 

The card in use in all thhe servers is an `LSI 2308-8i (9207-8i)`

### Drives:

All servers have `identical` setups (some differences like ram)

They are configured with the following:

- 1x 256G Boot SSD (rightmost populated drive port)
- 2x 4T Storage SSD (leftmost populated drive ports)

Else these servers have the following:

- 2x 6 core CPUs
- 168GB-192GB ram

## Network:

All the servers are configured identically network wise. However they have the following IPs

- Pluto: 158.27.6.30
- Skaftetryne: 158.37.6.12
- Bolivar: 158.37.6.21



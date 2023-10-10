+++
title = "High availability configuration"
description = "Simple walkthough of our current HA setup"
date = 2022-03-06T18:53:00+00:00
updated = 2022-03-06T18:53:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 1
draft = false
+++

## What is High Availability

High availability essentially describes a setup where if a node in proxmox goes down. Its services will be automatically spun up on another. 

A good video covering exactly our current setup can be found [here](https://www.youtube.com/watch?v=08b9DDJ_yf4&t=504s)

## Current configuration:

The current servers used for HA are Bolivar, Skaftetryne and Pluto. 

They are set up with a shared pool called `basseng`, this allows all vm drives to be stored with a copy on all the servers. This allows that once a Node goes down, the VMs will be spun up on other nodes  

### How the shared pool was set up

1. Enter a server in the cluster in Proxmox you want to use HA on
2. Click "ZFS" under the "Disks"-tab
3. Create a ZFS Pool
4. Name it (same thing for all servers in same pool)
5. Set RAID Level as "Mirror" (if using 3 or more drives use RaidZ or whatever fits)
6. For the "main"-pool for the cluster, make sure "Add Storage" is checked. On the other pools don't check off the "Add Storage"
7. Select all drives
8. Click "Create"
9. Repeat for all remaining servers in the pool, except unclick the `Add storage` checheckbox
10. After all ZFS pools are created, go to Storage & select the pool you created. Then add the remaining nodes inside "Nodes"

## A guide on how to enable HA for a VM can be found [here](/docs/instrukser/ha-setup/)

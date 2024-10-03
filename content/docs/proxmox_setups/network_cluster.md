+++
title = "Proxmox Network Cluster"
description = "Current setup of the proxmox networking cluster"
template = "docs/page.html"
sort_by = "weight"
weight = 1
draft = false
+++

# What?

This network consists of the two physical machines Netti and Letti.

These are to be used ONLY for vital networking services (DNS, Firewall, Proxies,
VPN)

If you want to host a general service, please put it on the regular proxmox
cluster

Each server uses 2x 1TB SSDs. These are set up in a ZFS mirror.

These will not use High Availability, so each service should be configured with
a failover (CARP/keepalived).

# Opnsese setup

Each of the hosts have a vm fw-{1-2}. These are the new firewalls we will be
running.

These run Opnsense (freeBSD) as a service and are currently accessible through
tailscale with fw-1.vpn.fribyte.no and fw-2.vpn.fribyte.no

The way these are set up is they have 4 network ports passed through to them
using PCI-passthrough in Proxmox.

## Wiring

These VMs are set up behind our current firewall (dole, ole), this is a
temporary setup for configuration.

# Diagram

![Diagram](/docs/proxmox_setups/proxmox_networking_cluster.png)

To edit the above diagram, use [draw.io](https://draw.io)

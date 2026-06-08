+++
title = "Network Overview"
description = "An attempt to get an overview of the machines on the network"
template = "docs/page.html"
sort_by = "weight"
weight = 10001
draft = false

[extra]
lang = "en"
translation = "docs/nettverk/nettverk-oversikt.md"
+++

# friByte subnet (No NAT)

## IPv4

- Network: 158.37.6.0/26
- Broadcast: 158.37.6.63
- Mask: 255.255.255.192
- Gateway: 158.37.6.33
- DNS servers: 158.37.6.21, 158.37.6.22, 1.1.1.1, 1.0.0.1

## IPv6

- Network: 2001:700:201:1::/64
- Gateway: 2001:700:201:1::
- DNS servers: 2001:700:201:1::53:2, 2001:700:201:1::53:1, 2606:4700:4700::1111, 2606:4700:4700::1001

| IPv4 (1-62) | IPv6                     | Name                           | Interface    | Comment                                 |
| ----------- | ------------------------ | ------------------------------ | ------------ | --------------------------------------- |
| 158.37.6.1  |                          | wiki.fribyte.no                |              | Zola wiki                               |
| 158.37.6.2  |                          |                                |              | (available)                             |
| 158.37.6.3  |                          |                                |              | (available)                             |
| 158.37.6.4  |                          |                                |              | (available)                             |
| 158.37.6.5  |                          | rf.uib.no                      |              | (customer)                              |
| 158.37.6.6  |                          |                                |              | (available)                             |
| 158.37.6.7  |                          | bstv.no                        |              | WordPress (customer)                    |
| 158.37.6.8  |                          | srib-skjema                    |              | LimeSurvey (customer)                   |
| 158.37.6.9  |                          | nat-public.kvarteret.no        | carp1        | Shared address                          |
| 158.37.6.10 |                          | dole.ss.uib.no                 | carp1        | Shared address                          |
| 158.37.6.11 |                          | klient.kvarteret.no            | carp1        | Shared address                          |
| 158.37.6.12 |                          | skaftetrynet.fribyte.no        |              | Proxmox node                            |
| 158.37.6.13 |                          | rootlinjeforening.no           |              | (customer)                              |
| 158.37.6.14 |                          | SUB-BSI.no                     |              | (customer)                              |
| 158.37.6.15 |                          | geo.uib.no, geo.fribyte.no     |              | (customer)                              |
| 158.37.6.16 |                          | srib-minecraft                 |              | (customer)                              |
| 158.37.6.17 |                          | mso.fribyte.no                 |              | (customer)                              |
| 158.37.6.18 |                          | haproxy1.ss.uib.no             |              | (dunstus) Old, use 48 and 49 instead    |
| 158.37.6.19 |                          | pengebingen                    |              | Docker environment (internal)           |
| 158.37.6.20 |                          |                                |              | (available)                             |
| 158.37.6.21 | 2001:700:201:1::d1       | ns1.fribyte.no                 |              | Nameserver                              |
| 158.37.6.22 | 2001:700:201:1::d2       | ns2.fribyte.no                 |              | Nameserver                              |
| 158.37.6.23 |                          | srib-radio                     |              | Docker environment (customer) (konrad)  |
| 158.37.6.24 | 2001:700:201:1::fd       |                                |              | Recursive nameserver                    |
| 158.37.6.25 |                          | btsi.no                        |              | (customer)                              |
| 158.37.6.26 |                          | skrue NFS + HTTP server        |              | Skrue                                   |
| 158.37.6.27 |                          | fergus.fribyte.no              | eno4         | Proxmox node                            |
| 158.37.6.28 |                          | Floating IP for lb-1 and lb-2  |              | shared between the lb-1 and lb-2 VMs    |
| 158.37.6.29 |                          | Mattermost                     |              | (internal)                              |
| 158.37.6.30 |                          | pluto.fribyte.no               |              | Proxmox node                            |
| 158.37.6.31 | 2001:700:201:1::2000     | gjertrud.fribyte.no            | vmbr0        | Proxmox                                 |
| 158.37.6.32 | 2001:700:201:1::2        | dole.ss.uib.no                 | em1          | Firewall + DHCP                         |
| 158.37.6.33 | 2001:700:201:1::0        | gw.ss.uib.no                   | carp1        | Shared address                          |
| 158.37.6.34 | 2001:700:201:1::1        | doffen.ss.uib.no               | em1          | Firewall + DHCP                         |
| 158.37.6.35 | 2001:700:201:1::3001     | bestemor - andeby.fribyte.no   | br0 (eth0)   | Former ganeti host + landing server     |
| 158.37.6.35 |                          | andeby.ss.uib.no               | br0:0 (eth0) | Points to bestemor                      |
| 158.37.6.36 | 2001:700:201:1::3002     | studvest                       | eth0         | Docker environment (customer)           |
| 158.37.6.37 |                          | db.fribyte.no                  |              | Database server for SRIB radio, bstv.no |
| 158.37.6.39 |                          | mso-staging-db                 |              | Database server for development of the new MSO |
| 158.37.6.40 | 2001:700:201:1::7001     | bingen.ss.uib.no               | eth0         | Backup machine                          |
| 158.37.6.41 |                          | galene.fribyte.no              |              | Galene streaming service                |
| 158.37.6.42 |                          | lytt.srib.no                   |              | SRIB AzuraCast                          |
| 158.37.6.43 |                          | skrue                          |              | Backup machine                          |
| 158.37.6.44 | 2001:700:201:1::3004     | solveig                        | eth0         | Ganeti host master                      |
| 158.37.6.45 | 2001:700:201:1::3003     | dunstus                        | eth0         | Ganeti host                             |
| 158.37.6.46 | 2001:700:201:1::7016     | magica.ss.uib.no               | eth0         | old internal server                     |
| 158.37.6.47 |                          |                                |              | (available)                             |
| 158.37.6.48 | 2001:700:201:1:5001::201 | load-balancer-1                | eth0         | Load balancer for Kubernetes cluster    |
| 158.37.6.49 | 2001:700:201:1:5001::202 | load-balancer-2                | eth0         | Load balancer for Kubernetes cluster    |
| 158.37.6.50 | 2001:700:201:1::7013     | kornelius.ss.uib.no            | eth0         | old monitoring - Munin                  |
| 158.37.6.51 |                          |                                |              | (available)                             |
| 158.37.6.52 | 2001:700:201:1::7010     | kladden.ss.uib.no              | eth0         | DNS server (solveig) (master)           |
| 158.37.6.53 | 2001:700:201:1::7018     | svartepetter.ss.uib.no         | eth0         | DNS server (dunstus) (slave)            |
| 158.37.6.54 |                          | Firewall CARP                  |              | OPNsense firewall CARP WAN gateway      |
| 158.37.6.55 |                          | fw-1                           |              | WAN address Firewall 1                  |
| 158.37.6.56 | 2001:700:201:1::107      | headscale.fribyte.no           |              | Headscale control server                |
| 158.37.6.57 |                          |                                |              |                                         |
| 158.37.6.58 |                          |                                |              |                                         |
| 158.37.6.59 |                          | fw-2                           |              | WAN address Firewall 2                  |
| 158.37.6.60 |                          |                                |              | (available)                             |
| 158.37.6.61 |                          |                                |              | (available)                             |
| 158.37.6.62 |                          | beszel.fribyte.no              |              | Beszel metrics                          |
| 158.37.6.63 |                          | Broadcast address              |              | (not reservable)                        |
| 158.37.6.64 |                          |                                |              | External IP                             |
| 158.37.6.65 |                          | dole                           |              | External IP                             |
| 158.37.6.66 |                          | dole                           |              | External IP                             |
| 158.37.6.67 |                          | doffen                         |              | External IP                             |

# OPNsense NAT network

This uses the subnet `10.0.0.0/24` for local addresses.

## IPv4

- Network: 10.0.0.0/24
- Broadcast: 10.0.0.255
- Mask: 255.255.255.0
- Gateway: 10.0.0.11
- DNS servers: 158.37.6.21, 158.37.6.22, 1.1.1.1, 1.0.0.1

| IPv4 (1-254) | IPv6 | Name            | Interface              | Comment              |
| ------------ | ---- | --------------- | ---------------------- | -------------------- |
| 10.0.0.1     |      | fw-1 (netti)    | bge0                   | LAN interface fw-1   |
| 10.0.0.2     |      | fw-2 (letti)    | bge0                   | LAN interface fw-2   |
| 10.0.0.11    |      | NAT gateway     |                        | OPNsense LAN gateway |
| 10.0.0.20    |      | Letti           | enp130s0f0             | Letti Proxmox host   |
| 10.0.0.21    |      | Netti           | enp132s0f0             | Netti Proxmox host   |
| 10.0.0.25    |      | Pluto           | enp7s0f1               | Netti Proxmox host   |
| 10.0.0.26    |      | Fergus          | enp7s0f1               | Netti Proxmox host   |
| 10.0.0.27    |      | Skaftetrynet    | eno1                   | Netti Proxmox host   |
| 10.0.0.70    |      | Raptus          | vmbrNAT (pluto)        | k3s master           |
| 10.0.0.71    |      | Petter          | vmbrNAT (Fergus)       | k3s master           |
| 10.0.0.72    |      | Hutre           | vmbrNAT (Skaftetrynet) | k3s master           |
| 10.0.0.80    |      | Lille-Hjelper-1 | vmbrNAT (Pluto)        | k3s slave            |
| 10.0.0.81    |      | Lille-Hjelper-2 | vmbrNAT (Fergus)       | k3s slave            |
| 10.0.0.82    |      | Lille-Hjelper-3 | vmbrNAT (Skaftetrynet) | k3s slave            |
| 10.0.0.100   |      | lb-1 (netti)    | vmbrNAT (netti)        | NAT TCP proxy        |
| 10.0.0.101   |      | lb-2 (letti)    | vmbrNAT (letti)        | NAT TCP proxy        |
|              |      |                 |                        |                      |

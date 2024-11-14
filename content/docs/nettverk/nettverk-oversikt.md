+++
title = "Nettverksoversikt"
description = "Et forsøk på å få oversikt over maskiner på nettverket"
template = "docs/page.html"
sort_by = "weight"
weight = 1
draft = false
+++

# Old network (No Nat)

## IPv4

- ip: 158.37.6.0/26
- broadcast: 158.37.6.63
- Mask: 255.255.255.192
- Gateway: 158.37.6.33
- DNS server: 158.37.6.52, 158.37.6.53, 1.1.1.1, 1.0.0.1

## IPv6

- ip: 2001:700:201:1::/64
- gateway: 2001:700:201:1::0
- dns server: 2001:700:201:1::53:2, 2001:700:201:1::53:1, 2606:4700:4700::1111,
  2606:4700:4700::1001

| IPv4        | IPv6                     | Navn                           | Interface    | Kommentar                               |
| ----------- | ------------------------ | ------------------------------ | ------------ | --------------------------------------- |
| 158.37.6.1  |                          | guffen                         |              | guffen self hosted actions runner       |
| 158.37.6.2  |                          |                                |              | (ledig)                                 |
| 158.37.6.3  |                          | workshop-website (temporarily) |              | Midlertidig brukt til website workshop  |
| 158.37.6.4  |                          | wiki.fribyte.no                |              | Zola wiki (konrad)                      |
| 158.37.6.5  |                          | rf.uib.no                      |              | (kunde)                                 |
| 158.37.6.6  |                          |                                |              | (ledig)                                 |
| 158.37.6.7  |                          | bstv.no                        |              | Wordpress (kunde) (konrad)              |
| 158.37.6.8  |                          | srib-skjema                    |              | (kunde) (konrad)                        |
| 158.37.6.9  |                          | nat-public.kvarteret.no        | carp1        | Felles addresse                         |
| 158.37.6.10 |                          | dole.ss.uib.no                 | carp1        | Felles addresse                         |
| 158.37.6.11 |                          | klient.kvarteret.no            | carp1        | Felles addresse                         |
| 158.37.6.12 |                          | Skaftetrynet.fribyte.no        |              | Proxmox node                            |
| 158.37.6.13 |                          | rootlinjeforening.no           |              | (kunde)                                 |
| 158.37.6.14 |                          | SUB-BSI.no                     |              | (kunde)                                 |
| 158.37.6.15 |                          | geo.uib.no, geo.fribyte.no     |              | (kunde)                                 |
| 158.37.6.16 |                          | srib-minecraft                 |              | (kunde) (konrad)                        |
| 158.37.6.17 |                          | mso.fribyte.no                 |              | (kunde)                                 |
| 158.37.6.18 |                          | haproxy1.ss.uib.no             |              | (dunstus) Gammel, bruk 48 og 49 cl      |
| 158.37.6.19 |                          | pengebingen                    |              | Docker-øko (intern) (konrad)            |
| 158.37.6.20 |                          | pluto.ss.uib.no                |              | Gammel webside server                   |
| 158.37.6.21 | 2001:700:201:1::d1       | ns1.fribyte.no                 |              | Navnetjener                             |
| 158.37.6.22 | 2001:700:201:1::d2       | ns2.fribyte.no                 |              | Navnetjener                             |
| 158.37.6.23 |                          | srib-radio                     |              | Docker-øko (kunde) (konrad)             |
| 158.37.6.24 | 2001:700:201:1::fd       |                                |              | Rekursiv navnetjener                    |
| 158.37.6.25 |                          | btsi.no                        |              | (kunde)                                 |
| 158.37.6.26 |                          | skrue NFS + http server        |              | Skrue                                   |
| 158.37.6.27 |                          | Fergus.fribyte.no              | eno4         | Proxmox node                            |
| 158.37.6.28 |                          | Flytende IP for lb-1 og lb-2   |              | delt mellom lb-1 og lb-2 VMer           |
| 158.37.6.29 |                          | Mattermost                     |              | (intern) (konrad)                       |
| 158.37.6.30 |                          | pluto.fribyte.no               |              | Proxmox node                            |
| 158.37.6.31 | 2001:700:201:1::2000     | gjertrud.fribyte.no            | vmbr0        | Proxmox                                 |
| 158.37.6.32 | 2001:700:201:1::2        | dole.ss.uib.no                 | em1          | Brannmur + DHCP                         |
| 158.37.6.33 | 2001:700:201:1::0        | gw.ss.uib.no                   | carp1        | Felles addresse                         |
| 158.37.6.34 | 2001:700:201:1::1        | doffen.ss.uib.no               | em1          | Brannmur + DHCP                         |
| 158.37.6.35 | 2001:700:201:1::3001     | bestemor - andeby.fribyte.no   | br0 (eth0)   | Tidligere ganeti host + landingsserver  |
| 158.37.6.35 |                          | andeby.ss.uib.no               | br0:0 (eth0) | Peker mot bestemor                      |
| 158.37.6.36 | 2001:700:201:1::3002     | studvest                       | eth0         | Docker-øko, (kunde) (konrad)            |
| 158.37.6.37 |                          | db.fribyte.no                  |              | Database server for SRIB radio, bstv.no |
| 158.37.6.39 |                          |                                |              | (ledig) (unknown service running on)    |
| 158.37.6.40 | 2001:700:201:1::7001     | bingen.ss.uib.no               | eth0         | Backup maskin                           |
| 158.37.6.41 |                          | galene.fribyte.no              |              | Galene streaming tjeneste               |
| 158.37.6.42 |                          |                                |              | (ledig)                                 |
| 158.37.6.43 |                          | skrue                          |              | Backup maskin                           |
| 158.37.6.44 | 2001:700:201:1::3004     | solveig                        | eth0         | ganeti host master                      |
| 158.37.6.45 | 2001:700:201:1::3003     | dunstus                        | eth0         | ganeti host                             |
| 158.37.6.46 | 2001:700:201:1::7016     | magica.ss.uib.no               | eth0         | gammel intern server                    |
| 158.37.6.47 |                          |                                |              | (ledig)                                 |
| 158.37.6.48 | 2001:700:201:1:5001::201 | load-balancer-1                | eth0         | Load balancer for kubernetes cluster    |
| 158.37.6.49 | 2001:700:201:1:5001::202 | load-balancer-2                | eth0         | Load balancer for kubernetes cluster    |
| 158.37.6.50 | 2001:700:201:1::7013     | kornelius.ss.uib.no            | eth0         | gammel overvåkning - Munin              |
| 158.37.6.51 | 2001:700:201:1::7000     | anton.ss.uib.no                | eth0         | gammel LDAP                             |
| 158.37.6.52 | 2001:700:201:1::7010     | kladden.ss.uib.no              | eth0         | DNS tjener (solveig) (master)           |
| 158.37.6.53 | 2001:700:201:1::7018     | svartepetter.ss.uib.no         | eth0         | DNS tjener (dunstus) (slave)            |
| 158.37.6.54 |                          | NEW FIREWALL CARP              |              | RESERVED                                |
| 158.37.6.55 |                          | WAN Firewall 1                 |              | RESERVED                                |
| 158.37.6.56 |                          | HEADSCALE                      |              |                                         |
| 158.37.6.57 |                          |                                |              |                                         |
| 158.37.6.58 |                          |                                |              |                                         |
| 158.37.6.59 |                          | WAN Firewall 2                 |              | RESERVED                                |
| 158.37.6.63 |                          | Broadcast                      | RESERVED     |                                         |
| 158.37.6.64 |                          |                                |              | (ledig)                                 |
| 158.37.6.65 |                          | dole                           |              | Ekstern ip                              |
| 158.37.6.66 |                          | dole                           |              | Ekstern ip                              |
| 158.37.6.67 |                          | doffen                         |              | Ekstern ip                              |

# New network (OpnSense NAT)

This uses the subnet `10.0.0.0/24` for local adresses

| IPV4      | IPV6 | Name            | Interface              | Comment              |
| --------- | ---- | --------------- | ---------------------- | -------------------- |
| 10.0.0.1  |      | fw-1 (netti)    | bge0                   | LAN interface fw-1   |
| 10.0.0.2  |      | fw-2            | bge0                   | Lan interface fw-2   |
| 10.0.0.11 |      | Gateway NAT     |                        | Opnsense LAN gateway |
| 10.0.0.20 |      | Letti           | enp130s0f0             | Letti Proxmox Host   |
| 10.0.0.21 |      | Netti           | enp132s0f0             | Netti Proxmox Host   |
| 10.0.0.25 |      | Pluto           | enp7s0f1               | Netti Proxmox Host   |
| 10.0.0.26 |      | Fergus          | enp7s0f1               | Netti Proxmox Host   |
| 10.0.0.27 |      | Skaftetrynet    | eno1                   | Netti Proxmox Host   |
| 10.0.0.70 |      | Raptus          | vmbrNAT (pluto)        | k3s Master           |
| 10.0.0.71 |      | Petter          | vmbrNAT (Fergus)       | k3s Master           |
| 10.0.0.72 |      | Hutre           | vmbrNAT (Skaftetrynet) | k3s Master           |
| 10.0.0.80 |      | Lille-Hjelper-1 | vmbrNAT (Pluto)        | k3s Slave            |
| 10.0.0.81 |      | Lille-Hjelper-2 | vmbrNAT (Fergus)       | k3s Slave            |
| 10.0.0.82 |      | Lille-Hjelper-3 | vmbrNAT (Skaftetrynet) | k3s Slave            |
|           |      |                 |                        |                      |
|           |      |                 |                        |                      |
|           |      |                 |                        |                      |

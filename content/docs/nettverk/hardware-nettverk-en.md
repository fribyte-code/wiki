+++
title = "Network hardware"
description = "A review of the pf rule set in production"
template = "docs/page.html"
sort_by = "weight"
weight = 10001
draft = false
toc = true

[extra]
lang = "en"
translation = "docs/nettverk/hardware-nettverk.md"
+++

# Information is imported from the old wiki. It may be and probably is outdated!

<!--

## Contents

 - [<span class="tocnumber">1</span> <span class="toctext">Network</span>](#Network)
  - [<span class="tocnumber">1.1</span> <span class="toctext">Cable labeling</span>](#Cable_labeling)
- [<span class="tocnumber">2</span> <span class="toctext">VLAN</span>](#VLAN)
- [<span class="tocnumber">3</span> <span class="toctext">Network overview / VLAN</span>](#Network_overview_VLAN)
- [<span class="tocnumber">4</span> <span class="toctext">Routing overview</span>](#Routing_overview)
- [<span class="tocnumber">5</span> <span class="toctext">Subnets and IPs</span>](#Subnets_and_IPs)
  - [<span class="tocnumber">5.1</span> <span class="toctext">Uplink network to UiB</span>](#Uplink_network_to_UiB)
  - [<span class="tocnumber">5.2</span> <span class="toctext">Server network (ss.uib.no)</span>](#Server_network)
  - [<span class="tocnumber">5.3</span> <span class="toctext">DRBD sync network</span>](#DRBD_sync_network)
  - [<span class="tocnumber">5.4</span> <span class="toctext">Kvarteret cash register network</span>](#Kvarteret_cash_register_network)
  - [<span class="tocnumber">5.5</span> <span class="toctext">Kvarteret client network</span>](#Kvarteret_client_network)
  - [<span class="tocnumber">5.6</span> <span class="toctext">Kvarteret DORGe network</span>](#Kvarteret_DORGe-network)
  - [<span class="tocnumber">5.7</span> <span class="toctext">Kvarteret Internet only / reserve network</span>](#Kvarteret_Internet_only/reserve_network)
  - [<span class="tocnumber">5.8</span> <span class="toctext">Management network</span>](#Management_network)
  - [<span class="tocnumber">5.9</span> <span class="toctext">SriB Server</span>](#SriB_Server)
  - [<span class="tocnumber">5.10</span> <span class="toctext">SriB Client</span>](#SriB_Client)
  - [<span class="tocnumber">5.11</span> <span class="toctext">Studvest</span>](#Studvest)
  - [<span class="tocnumber">5.12</span> <span class="toctext">BSTV</span>](#BSTV)
  - [<span class="tocnumber">5.13</span> <span class="toctext">pfsync</span>](#pfsync) -->

# Network

## Cable labeling

To make things a bit more organized, cables should be color-labeled with tape
(stored in the drawer in the server room).

<table class="wikitable" cellpadding="12">

<tbody>

<tr>

<th>Color</th>

<th>Meaning</th>

</tr>

<tr>

<td>red</td>

<td>Uplink to UiB</td>

</tr>

<tr>

<td>blue</td>

<td>Server network</td>

</tr>

<tr>

<td>yellow-green</td>

<td>drbd sync</td>

</tr>

<tr>

<td>black</td>

<td>management network</td>

</tr>

</tbody>

</table>

# VLAN

VLAN numbers are usually chosen according to the following pattern:

- 0-1: Standard networks found on all kinds of network equipment. Should not be
  used in practice because default configurations often use these and can
  therefore interfere with things.
- 2-99: Server networks (only in server rooms, have public IP addresses or
  internal ones outside 10.250)
- 100-199: Internal networks (primarily for Kvarteret) with IPv4 addresses
  of the form 10.250.xx, where xx corresponds to VLAN 1xx
- 200-255: Internal special/server networks, often with IPv4 addresses of the
  form 10.250.2xx, where xx corresponds to VLAN 2xx
- 300-399: Various pseudo-networks for assorted purposes (e.g. PacketFence)

<table class="wikitable" cellpadding="12">

<tbody>

<tr>

<th>VLAN number</th>

<th>What it is used for</th>

<th>Connected to the firewalls?</th>

</tr>

<tr>

<td>1</td>

<td>Default VLAN</td>

<td>no</td>

</tr>

<tr>

<td>2</td>

<td>Server network</td>

<td>yes</td>

</tr>

<tr>

<td>3</td>

<td>DRBD</td>

<td>no</td>

</tr>

<tr>

<td>10</td>

<td>SriB Server</td>

<td>yes</td>

</tr>

<tr>

<td>38</td>

<td>VPN DAK</td>

<td>no</td>

</tr>

<tr>

<td>101</td>

<td>Kvarteret cash register</td>

<td>yes</td>

</tr>

<tr>

<td>110</td>

<td>Kvarteret client</td>

<td>yes</td>

</tr>

<tr>

<td>170</td>

<td>Kvarteret DORG</td>

<td>no</td>

</tr>

<tr>

<td>180</td>

<td>Kvarteret Internet only / reserve</td>

<td>yes</td>

</tr>

<tr>

<td>200</td>

<td>Management network</td>

<td>yes</td>

</tr>

<tr>

<td>302</td>

<td>PacketFence registration</td>

<td>no</td>

</tr>

<tr>

<td>303</td>

<td>PacketFence isolation</td>

<td>no</td>

</tr>

<tr>

<td>304</td>

<td>PacketFence MAC discovery</td>

<td>no</td>

</tr>

<tr>

<td>305</td>

<td>PacketFence IP telephony</td>

<td>no</td>

</tr>

<tr>

<td>306</td>

<td>PacketFence firewall-based filtering</td>

<td>no</td>

</tr>

<tr>

<td>412</td>

<td>SriB Client</td>

<td>yes</td>

</tr>

<tr>

<td>420</td>

<td>Studvest</td>

<td>yes</td>

</tr>

<tr>

<td>430</td>

<td>BSTV</td>

<td>yes</td>

</tr>

</tbody>

</table>

# Network overview VLAN

![Fribyte-network.png](/1500px-Fribyte-network.png) Dotted links are
management connections from ipmi / bmc cards.

# Routing overview

![Fribyte-routing.png](/Fribyte-routing.png) The IPs under the different
networks are the shared address the firewalls share.

# Subnets and IPs

## Uplink_network_to_UiB

**IPv4**

- ip: 158.37.2.16/29
- Mask: 255.255.255.248
- Gateway: 158.37.2.17

**IPv6**

- ip: 2001:700:201:f001::/64
- gateway: 2001:700:201:F001::1

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ipv4</th>

<th>ipv6</th>

<th>name / machine</th>

<th>interface</th>

<th>note</th>

</tr>

<tr>

<td>158.37.2.18</td>

<td>2001:700:201:f001::2</td>

<td>uplink.ss.uib.no</td>

<td>carp2</td>

<td>uplink to UiB (shared address)</td>

</tr>

<tr>

<td>158.37.2.19</td>

<td>2001:700:201:f001::3</td>

<td>dole.uplink.ss.uib.no</td>

<td>em2</td>

<td>uplink to UiB</td>

</tr>

<tr>

<td>158.37.2.20</td>

<td>2001:700:201:f001::4</td>

<td>doffen.uplink.ss.uib.no</td>

<td>em2</td>

<td>uplink to UiB</td>

</tr>

</tbody>

</table>

### Server network

**IPv4**

- ip: 158.37.6.0/26
- broadcast: 158.37.6.63
- Mask: 255.255.255.192
- Gateway: 158.37.6.33
- DNS server: 158.37.6.52, 158.37.6.53

**IPv6**

- ip: 2001:700:201:1::/64
- gateway: 2001:700:201:1::0
- dns server 2001:700:201:1::53:2, 2001:700.201:1::53:1

See [Network overview](@/docs/nettverk/nettverk-oversikt.md) for the inventory
list of internal ip addresses.

### DRBD_sync_network

**IPv4**

- ip: 10.10.10.0/24
- Mask: 255.255.255.0

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>name</th>

<th>interface</th>

<th>note</th>

</tr>

<tr>

<td>10.10.10.10</td>

<td>bestefar</td>

<td>eth1</td>

<td>drbd</td>

</tr>

<tr>

<td>10.10.10.11</td>

<td>bestemor</td>

<td>eth1</td>

<td>drbd</td>

</tr>

<tr>

<td>10.10.10.12</td>

<td>hetti</td>

<td>eth1</td>

<td>drbd</td>

</tr>

<tr>

<td>10.10.10.13</td>

<td>netti</td>

<td>eth1</td>

<td>drbd</td>

</tr>

<tr>

<td>10.10.10.14</td>

<td>letti</td>

<td>eth1</td>

<td>drbd</td>

</tr>

</tbody>

</table>

### Kvarteret_cash_register_network

(No longer in use)

**IPv4**

- ip: 10.250.1.0/24
- mask: 255.255.255.0
- gateway: 10.250.1.1
- dns: 158.37.6.53, 158.37.6.52

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>name</th>

<th>interface</th>

<th>note</th>

</tr>

<tr>

<td>10.250.1.1</td>

<td>uplink</td>

<td>carp3</td>

<td>Cash register network (shared address)</td>

</tr>

<tr>

<td>10.250.1.2</td>

<td>dole</td>

<td>vlan1</td>

<td>Cash register network</td>

</tr>

<tr>

<td>10.250.1.3</td>

<td>doffen</td>

<td>vlan1</td>

<td>Cash register network</td>

</tr>

</tbody>

</table>

### Kvarteret_client_network

**IPv4**

- ip: 10.250.10.0/23
- Mask: 255.255.254.0
- gateway: 10.250.10.1
- dns: 158.37.6.50

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>name</th>

<th>interface</th>

<th>note</th>

</tr>

<tr>

<td>10.250.10.1</td>

<td>uplink</td>

<td>carp4</td>

<td>Client network (shared address)</td>

</tr>

<tr>

<td>10.250.10.11</td>

<td>dole</td>

<td>vlan10</td>

<td>Client network</td>

</tr>

<tr>

<td>10.250.10.12</td>

<td>doffen</td>

<td>vlan10</td>

<td>Client network</td>

</tr>

</tbody>

</table>

### Kvarteret_DORGe-network

**IPv4**

- ip: 10.250.70.0/24
- Mask: 255.255.255.0
- gateway: 10.250.70.1
- dns: 158.37.6.50

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>name</th>

<th>interface</th>

<th>note</th>

</tr>

<tr>

<td>10.250.70.1</td>

<td>uplink</td>

<td>carp170</td>

<td>DORGe network (shared address)</td>

</tr>

<tr>

<td>10.250.70.2</td>

<td>dole</td>

<td>vlan170</td>

<td>DORGe network</td>

</tr>

<tr>

<td>10.250.70.3</td>

<td>doffen</td>

<td>vlan170</td>

<td>DORGe network</td>

</tr>

</tbody>

</table>

### Kvarteret_Internet_only/reserve_network

**IPv4**

- ip: 10.250.80.0/24
- Mask: 255.255.255.0
- gateway: 10.250.80.1
- dns: 158.37.6.50

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>name</th>

<th>interface</th>

<th>note</th>

</tr>

<tr>

<td>10.250.80.1</td>

<td>uplink</td>

<td>carp180</td>

<td>Internet-only / reserve network (shared address)</td>

</tr>

<tr>

<td>10.250.80.2</td>

<td>dole</td>

<td>vlan180</td>

<td>Internet-only / reserve network</td>

</tr>

<tr>

<td>10.250.80.3</td>

<td>doffen</td>

<td>vlan180</td>

<td>Internet-only / reserve network</td>

</tr>

</tbody>

</table>

### Management_network

**IPv4**

- ip: 10.250.200.0/24
- Mask: 255.255.255.0
- gateway: 10.250.200.1

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>name</th>

<th>interface</th>

<th>note</th>

</tr>

<tr>

<td>10.250.200.1</td>

<td>uplink.oob.fribyte.no</td>

<td>carp5</td>

<td>shared address</td>

</tr>

<tr>

<td>10.250.200.2</td>

<td>dole.oob.fribyte.no</td>

<td>vlan200</td>

<td></td>

</tr>

<tr>

<td>10.250.200.3</td>

<td>doffen.oob.fribyte.no</td>

<td>vlan200</td>

<td></td>

</tr>

<tr>

<td>10.250.200.10</td>

<td>ssw1.oob.fribyte.no</td>

<td></td>

<td>Switch in the SV server room (top left rack) `<c>telnet ssw1.oob.fribyte.no</c>` to view the router configuration</td>

</tr>

<tr>

<td>10.250.200.11</td>

<td>ssw2.oob.fribyte.no</td>

<td></td>

<td>Switch in the SV server room (bottom right rack)</td>

</tr>

<tr>

<td>10.250.200.20</td>

<td>oksw1.oob.fribyte.no</td>

<td></td>

<td>Retired, formerly Switch 1 Kvarteret (Olav Kyrres gate)</td>

</tr>

<tr>

<td>10.250.200.21</td>

<td>oksw2.oob.fribyte.no</td>

<td></td>

<td>Retired, formerly Switch 2 Kvarteret (Olav Kyrres gate)</td>

</tr>

<tr>

<td>10.250.200.22</td>

<td>oksw3.oob.fribyte.no</td>

<td></td>

<td>Retired, formerly Switch 3 Kvarteret (Olav Kyrres gate)</td>

</tr>

<tr>

<td>10.250.200.25</td>

<td>fwsw1.oob.fribyte.no</td>

<td></td>

<td>Switch at Kvarteret (Fosswinkels gate)</td>

</tr>

<tr>

<td>10.250.200.30</td>

<td>sssw1.oob.fribyte.no</td>

<td></td>

<td>Switch 1 Student Center (SriB)</td>

</tr>

<tr>

<td>10.250.200.31</td>

<td>sssw2.oob.fribyte.no</td>

<td></td>

<td>Switch 2 Student Center (Studvest, BSTV)</td>

</tr>

<tr>

<td>10.250.200.50</td>

<td>bestefar.oob.fribyte.no</td>

<td></td>

<td>ipmi</td>

</tr>

<tr>

<td>10.250.200.51</td>

<td>hetti.oob.fribyte.no</td>

<td></td>

<td>ipmi</td>

</tr>

<tr>

<td>10.250.200.52</td>

<td>netti.oob.fribyte.no</td>

<td></td>

<td>ipmi</td>

</tr>

<tr>

<td>10.250.200.53</td>

<td>letti.oob.fribyte.no</td>

<td></td>

<td>ipmi</td>

</tr>

<tr>

<td>10.250.200.54</td>

<td>bestemor.oob.fribyte.no</td>

<td></td>

<td>ipmi</td>

</tr>

<tr>

<td>10.250.200.55</td>

<td>della.oob.fribyte.no</td>

<td></td>

<td>nic1</td>

</tr>

<tr>

<td>10.250.200.56</td>

<td>managment.oob.fribyte.no</td>

<td></td>

<td>eth0 (della)</td>

</tr>

<tr>

<td>10.250.200.57</td>

<td>ole.oob.fribyte.no</td>

<td></td>

<td>nic2</td>

</tr>

<tr>

<td>10.250.200.58</td>

<td>bgjengen.oob.fribyte.no</td>

<td></td>

<td>ipmi</td>

</tr>

</tbody>

</table>

### SriB_Server

**IPv4**

- ip: 158.37.6.64/26
- Mask: 255.255.255.192
- gateway: 158.37.6.65
- dns: 158.37.6.50

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>name</th>

<th>interface</th>

<th>note</th>

</tr>

<tr>

<td>10.250.112.1</td>

<td>uplink</td>

<td>carp6</td>

<td>SRiB Server (shared address)</td>

</tr>

<tr>

<td>10.250.112.2</td>

<td>dole</td>

<td>vlan10</td>

<td>SRiB Server</td>

</tr>

<tr>

<td>10.250.112.3</td>

<td>doffen</td>

<td>vlan10</td>

<td>SRiB Server</td>

</tr>

</tbody>

</table>

### SriB_Client

**IPv4**

- ip: 10.250.112.0/24
- Mask: 255.255.255.0
- gateway: 10.250.112.1
- dns: 158.37.6.50

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>name</th>

<th>interface</th>

<th>note</th>

</tr>

<tr>

<td>10.250.112.1</td>

<td>uplink</td>

<td>carp7</td>

<td>SRiB Client (shared address)</td>

</tr>

<tr>

<td>10.250.112.2</td>

<td>dole</td>

<td>vlan412</td>

<td>SRiB Client / reserve network</td>

</tr>

<tr>

<td>10.250.112.3</td>

<td>doffen</td>

<td>vlan412</td>

<td>SRiB Client / reserve network</td>

</tr>

</tbody>

</table>

### Studvest

**IPv4**

- ip: 10.250.120.0/24
- Mask: 255.255.255.0
- gateway: 10.250.120.1
- dns: 158.37.6.50

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>name</th>

<th>interface</th>

<th>note</th>

</tr>

<tr>

<td>10.250.120.1</td>

<td>uplink</td>

<td>carp10</td>

<td>Studvest (shared address)</td>

</tr>

<tr>

<td>10.250.120.2</td>

<td>dole</td>

<td>vlan420</td>

<td>Studvest / reserve network</td>

</tr>

<tr>

<td>10.250.120.3</td>

<td>doffen</td>

<td>vlan420</td>

<td>Studvest / reserve network</td>

</tr>

</tbody>

</table>

### BSTV

**IPv4**

- ip: 10.250.130.0/24
- Mask: 255.255.255.0
- gateway: 10.250.130.1
- dns: 158.37.6.50

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>name</th>

<th>interface</th>

<th>note</th>

</tr>

<tr>

<td>10.250.130.1</td>

<td>uplink</td>

<td>carp11</td>

<td>BSTV (shared address)</td>

</tr>

<tr>

<td>10.250.130.2</td>

<td>dole</td>

<td>vlan430</td>

<td>BSTV / reserve network</td>

</tr>

<tr>

<td>10.250.130.3</td>

<td>doffen</td>

<td>vlan430</td>

<td>BSTV / reserve network</td>

</tr>

</tbody>

</table>

### pfsync

**IPv4**

- ip: 10.7.7.0/24
- Mask: 255.255.255.0

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>name</th>

<th>interface</th>

<th>note</th>

</tr>

<tr>

<td>10.7.7.1</td>

<td>dole</td>

<td>bge0</td>

<td>pfsync (also syncs DHCP)</td>

</tr>

<tr>

<td>10.7.7.2</td>

<td>doffen</td>

<td>bge0</td>

<td>pfsync (also syncs DHCP)</td>

</tr>

</tbody>

</table>

</div>

Retrieved from
«[https://wiki.uib.no/fribyte/index.php?title=Hardware:nettverk&oldid=1904](https://wiki.uib.no/fribyte/index.php?title=Hardware:nettverk&oldid=1904)»

</div>

+++
title = "Hardware nettverk"
description = "En gjennomgang av pf-regelsettet som er i produksjon"
template = "docs/page.html"
sort_by = "weight"
weight = 1
draft = false
toc = true
+++

# Informasjon er importert fra gammel wiki. Kan og er nok utdatert!

<!--

## Innhold

 - [<span class="tocnumber">1</span> <span class="toctext">Nettverk</span>](#Nettverk)
  - [<span class="tocnumber">1.1</span> <span class="toctext">Kabelmerking</span>](#Kabelmerking)
- [<span class="tocnumber">2</span> <span class="toctext">VLAN</span>](#VLAN)
- [<span class="tocnumber">3</span> <span class="toctext">Nettverksoversikt / VLAN</span>](#Nettverksoversikt_VLAN)
- [<span class="tocnumber">4</span> <span class="toctext">Routingoversikt</span>](#Rutingoversikt)
- [<span class="tocnumber">5</span> <span class="toctext">Subnett og IPer</span>](#Subnett_og_IPer)
  - [<span class="tocnumber">5.1</span> <span class="toctext">Uplink nett mot UiB</span>](#Uplink_nett_mot_UiB)
  - [<span class="tocnumber">5.2</span> <span class="toctext">Server nettverk (ss.uib.no)</span>](#Servernettverk)
  - [<span class="tocnumber">5.3</span> <span class="toctext">DRBD sync nett</span>](#DRBD_sync_nett)
  - [<span class="tocnumber">5.4</span> <span class="toctext">Kvarteret Kasse nett</span>](#Kvarteret_Kasse_nett)
  - [<span class="tocnumber">5.5</span> <span class="toctext">Kvarteret Klient nett</span>](#Kvarteret_Klient_nett)
  - [<span class="tocnumber">5.6</span> <span class="toctext">Kvarteret DORGe-nett</span>](#Kvarteret_DORGe-nett)
  - [<span class="tocnumber">5.7</span> <span class="toctext">Kvarteret Bare Internett / resevere nett</span>](#Kvarteret_Bare_Internett/resevere_nett)
  - [<span class="tocnumber">5.8</span> <span class="toctext">Management nett</span>](#Management_nett)
  - [<span class="tocnumber">5.9</span> <span class="toctext">SriB Server</span>](#SriB_Server)
  - [<span class="tocnumber">5.10</span> <span class="toctext">SriB Klient</span>](#SriB_Klient)
  - [<span class="tocnumber">5.11</span> <span class="toctext">Studvest</span>](#Studvest)
  - [<span class="tocnumber">5.12</span> <span class="toctext">BSTV</span>](#BSTV)
  - [<span class="tocnumber">5.13</span> <span class="toctext">pfsync</span>](#pfsync) -->

# Nettverk

## Kabelmerking

For å gjøre ting litt mer oversiktlig skal kabler bli fargemerket med tape
(ligger i skuffen på serverrommet).

<table class="wikitable" cellpadding="12">

<tbody>

<tr>

<th>Farge</th>

<th>Betydning</th>

</tr>

<tr>

<td>rød</td>

<td>Uplink til UiB</td>

</tr>

<tr>

<td>blå</td>

<td>Server nett</td>

</tr>

<tr>

<td>gul-grønn</td>

<td>drbd sync</td>

</tr>

<tr>

<td>svart</td>

<td>management nett</td>

</tr>

</tbody>

</table>

# VLAN

VLAN-nummer blir vanlgvis valgt etter følgende mønster:

- 0-1: Standard-nettverk som finnes på alt slags nettverksutstyr. Bør ikke
  brukes i praksis fordi standard-konfigurasjoner ofte bruker disse og dermed
  kan forstyrre ting.
- 2-99: Server-nettverk (kun på serverrom, har offentlige IP-adresser eller
  interne utenfor 10.250)
- 100-199: Interne nettverk (i utgangspunktet for Kvarteret) med IPv4-adresser
  på formen 10.250.xx, der xx tilsvarer VLAN 1xx
- 200-255: Interne spesial/server-nettverk, gjerne med IPv4-adresser på formen
  10.250.2xx, der xx svarer til VLAN 2xx
- 300-399: Diverse pseudo-nettverk for ymse formål (f.eks. PacketFence)

<table class="wikitable" cellpadding="12">

<tbody>

<tr>

<th>VLAN nummer</th>

<th>Hva det er brukt til</th>

<th>Koblet til brannmurene?</th>

</tr>

<tr>

<td>1</td>

<td>Default VLAN</td>

<td>nei</td>

</tr>

<tr>

<td>2</td>

<td>Server nett</td>

<td>ja</td>

</tr>

<tr>

<td>3</td>

<td>DRBD</td>

<td>nei</td>

</tr>

<tr>

<td>10</td>

<td>SriB Server</td>

<td>ja</td>

</tr>

<tr>

<td>38</td>

<td>VPN DAK</td>

<td>nei</td>

</tr>

<tr>

<td>101</td>

<td>Kvarteret Kasse</td>

<td>ja</td>

</tr>

<tr>

<td>110</td>

<td>Kvarteret Klient</td>

<td>ja</td>

</tr>

<tr>

<td>170</td>

<td>Kvarteret DORG</td>

<td>nei</td>

</tr>

<tr>

<td>180</td>

<td>Kvarteret Bare Internett / resevere</td>

<td>ja</td>

</tr>

<tr>

<td>200</td>

<td>Management nett</td>

<td>ja</td>

</tr>

<tr>

<td>302</td>

<td>PacketFence registrering</td>

<td>nei</td>

</tr>

<tr>

<td>303</td>

<td>PacketFence isolasjon</td>

<td>nei</td>

</tr>

<tr>

<td>304</td>

<td>PacketFence MAC-oppdagelse</td>

<td>nei</td>

</tr>

<tr>

<td>305</td>

<td>PacketFence IP-telefoni</td>

<td>nei</td>

</tr>

<tr>

<td>306</td>

<td>PacketFence brannmur-basert filtrering</td>

<td>nei</td>

</tr>

<tr>

<td>412</td>

<td>SriB Klient</td>

<td>ja</td>

</tr>

<tr>

<td>420</td>

<td>Studvest</td>

<td>ja</td>

</tr>

<tr>

<td>430</td>

<td>BSTV</td>

<td>ja</td>

</tr>

</tbody>

</table>

# Nettverksoversikt VLAN

![Fribyte-network.png](/1500px-Fribyte-network.png) Stiplede linker er
management oppkoblinger fra ipmi / bmc kort.

# Rutingoversikt

![Fribyte-routing.png](/Fribyte-routing.png) IPer under de forskjellige nettene
er felles addressen brannmurene deler.

# Subnett og IPer

## Uplink_nett_mot_UiB

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

<th>navn / maskin</th>

<th>interface</th>

<th>notis</th>

</tr>

<tr>

<td>158.37.2.18</td>

<td>2001:700:201:f001::2</td>

<td>uplink.ss.uib.no</td>

<td>carp2</td>

<td>uplink til UiB (felles addresse)</td>

</tr>

<tr>

<td>158.37.2.19</td>

<td>2001:700:201:f001::3</td>

<td>dole.uplink.ss.uib.no</td>

<td>em2</td>

<td>uplink til UiB</td>

</tr>

<tr>

<td>158.37.2.20</td>

<td>2001:700:201:f001::4</td>

<td>doffen.uplink.ss.uib.no</td>

<td>em2</td>

<td>uplink til UiB</td>

</tr>

</tbody>

</table>

### Servernettverk

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

Se [Nettverksoversikt](@/docs/nettverk/nettverk-oversikt.md) for inventarliste
over interne ip-addresser.

### DRBD_sync_nett

**IPv4**

- ip: 10.10.10.0/24
- Mask: 255.255.255.0

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>navn</th>

<th>interface</th>

<th>notis</th>

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

### Kvarteret_Kasse_nett

(Ikke i bruk lengre)

**IPv4**

- ip: 10.250.1.0/24
- mask: 255.255.255.0
- gateway: 10.250.1.1
- dns: 158.37.6.53, 158.37.6.52

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>navn</th>

<th>interface</th>

<th>notis</th>

</tr>

<tr>

<td>10.250.1.1</td>

<td>uplink</td>

<td>carp3</td>

<td>Kasse nett (felles addresse)</td>

</tr>

<tr>

<td>10.250.1.2</td>

<td>dole</td>

<td>vlan1</td>

<td>Kasse nett</td>

</tr>

<tr>

<td>10.250.1.3</td>

<td>doffen</td>

<td>vlan1</td>

<td>Kasse nett</td>

</tr>

</tbody>

</table>

### Kvarteret_Klient_nett

**IPv4**

- ip: 10.250.10.0/23
- Mask: 255.255.254.0
- gateway: 10.250.10.1
- dns: 158.37.6.50

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>navn</th>

<th>interface</th>

<th>notis</th>

</tr>

<tr>

<td>10.250.10.1</td>

<td>uplink</td>

<td>carp4</td>

<td>Klient nett (felles addresse)</td>

</tr>

<tr>

<td>10.250.10.11</td>

<td>dole</td>

<td>vlan10</td>

<td>Klient nett</td>

</tr>

<tr>

<td>10.250.10.12</td>

<td>doffen</td>

<td>vlan10</td>

<td>Klient nett</td>

</tr>

</tbody>

</table>

### Kvarteret_DORGe-nett

**IPv4**

- ip: 10.250.70.0/24
- Mask: 255.255.255.0
- gateway: 10.250.70.1
- dns: 158.37.6.50

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>navn</th>

<th>interface</th>

<th>notis</th>

</tr>

<tr>

<td>10.250.70.1</td>

<td>uplink</td>

<td>carp170</td>

<td>DORGe-nett (felles addresse)</td>

</tr>

<tr>

<td>10.250.70.2</td>

<td>dole</td>

<td>vlan170</td>

<td>DORGe-nett</td>

</tr>

<tr>

<td>10.250.70.3</td>

<td>doffen</td>

<td>vlan170</td>

<td>DORGe-nett</td>

</tr>

</tbody>

</table>

### Kvarteret_Bare_Internett/resevere_nett

**IPv4**

- ip: 10.250.80.0/24
- Mask: 255.255.255.0
- gateway: 10.250.80.1
- dns: 158.37.6.50

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>navn</th>

<th>interface</th>

<th>notis</th>

</tr>

<tr>

<td>10.250.80.1</td>

<td>uplink</td>

<td>carp180</td>

<td>Bare Internett / reserve nett (felles addresse)</td>

</tr>

<tr>

<td>10.250.80.2</td>

<td>dole</td>

<td>vlan180</td>

<td>Bare Internett / reserve nett</td>

</tr>

<tr>

<td>10.250.80.3</td>

<td>doffen</td>

<td>vlan180</td>

<td>Bare Internett / reserve nett</td>

</tr>

</tbody>

</table>

### Management_nett

**IPv4**

- ip: 10.250.200.0/24
- Mask: 255.255.255.0
- gateway: 10.250.200.1

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>navn</th>

<th>interface</th>

<th>notis</th>

</tr>

<tr>

<td>10.250.200.1</td>

<td>uplink.oob.fribyte.no</td>

<td>carp5</td>

<td>felles addresse</td>

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

<td>Switch switch på serverrommet SV (øverst venstre rack) `<c>telnet ssw1.oob.fribyte.no</c>` for å se på ruterkonfigurasjon</td>

</tr>

<tr>

<td>10.250.200.11</td>

<td>ssw2.oob.fribyte.no</td>

<td></td>

<td>Switch switch på serverrommet SV (nede høyre rack)</td>

</tr>

<tr>

<td>10.250.200.20</td>

<td>oksw1.oob.fribyte.no</td>

<td></td>

<td>Pensjonert, tidligere Switch 1 Kvarteret (Olav Kyrres gate)</td>

</tr>

<tr>

<td>10.250.200.21</td>

<td>oksw2.oob.fribyte.no</td>

<td></td>

<td>Pensjonert, tidligere Switch 2 Kvarteret (Olav Kyrres gate)</td>

</tr>

<tr>

<td>10.250.200.22</td>

<td>oksw3.oob.fribyte.no</td>

<td></td>

<td>Pensjonert, tidligere Switch 3 Kvarteret (Olav Kyrres gate)</td>

</tr>

<tr>

<td>10.250.200.25</td>

<td>fwsw1.oob.fribyte.no</td>

<td></td>

<td>Switch Kvarteret (Fosswinkels gate)</td>

</tr>

<tr>

<td>10.250.200.30</td>

<td>sssw1.oob.fribyte.no</td>

<td></td>

<td>Switch 1 Studentsenteret (SriB)</td>

</tr>

<tr>

<td>10.250.200.31</td>

<td>sssw2.oob.fribyte.no</td>

<td></td>

<td>Switch 2 Studentsenteret (Studvest, BSTV)</td>

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

<th>navn</th>

<th>interface</th>

<th>notis</th>

</tr>

<tr>

<td>10.250.112.1</td>

<td>uplink</td>

<td>carp6</td>

<td>SRiB Server (felles addresse)</td>

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

### SriB_Klient

**IPv4**

- ip: 10.250.112.0/24
- Mask: 255.255.255.0
- gateway: 10.250.112.1
- dns: 158.37.6.50

<table class="wikitable" cellpadding="12" align="top">

<tbody>

<tr>

<th>ip</th>

<th>navn</th>

<th>interface</th>

<th>notis</th>

</tr>

<tr>

<td>10.250.112.1</td>

<td>uplink</td>

<td>carp7</td>

<td>SRiB Klient (felles addresse)</td>

</tr>

<tr>

<td>10.250.112.2</td>

<td>dole</td>

<td>vlan412</td>

<td>SRiB Klient / reserve nett</td>

</tr>

<tr>

<td>10.250.112.3</td>

<td>doffen</td>

<td>vlan412</td>

<td>SRiB Klient / reserve nett</td>

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

<th>navn</th>

<th>interface</th>

<th>notis</th>

</tr>

<tr>

<td>10.250.120.1</td>

<td>uplink</td>

<td>carp10</td>

<td>Studvest (felles addresse)</td>

</tr>

<tr>

<td>10.250.120.2</td>

<td>dole</td>

<td>vlan420</td>

<td>Studvest / reserve nett</td>

</tr>

<tr>

<td>10.250.120.3</td>

<td>doffen</td>

<td>vlan420</td>

<td>Studvest / reserve nett</td>

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

<th>navn</th>

<th>interface</th>

<th>notis</th>

</tr>

<tr>

<td>10.250.130.1</td>

<td>uplink</td>

<td>carp11</td>

<td>BSTV (felles addresse)</td>

</tr>

<tr>

<td>10.250.130.2</td>

<td>dole</td>

<td>vlan430</td>

<td>BSTV / reserve nett</td>

</tr>

<tr>

<td>10.250.130.3</td>

<td>doffen</td>

<td>vlan430</td>

<td>BSTV / reserve nett</td>

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

<th>navn</th>

<th>interface</th>

<th>notis</th>

</tr>

<tr>

<td>10.7.7.1</td>

<td>dole</td>

<td>bge0</td>

<td>pfsync (syncer også DHCP)</td>

</tr>

<tr>

<td>10.7.7.2</td>

<td>doffen</td>

<td>bge0</td>

<td>pfsync (syncer også DHCP)</td>

</tr>

</tbody>

</table>

</div>

Hentet fra
«[https://wiki.uib.no/fribyte/index.php?title=Hardware:nettverk&oldid=1904](https://wiki.uib.no/fribyte/index.php?title=Hardware:nettverk&oldid=1904)»

</div>

+++
title = "DNS"
description = "Information about friByte's DNS service"
template = "docs/page.html"
sort_by = "weight"
weight = 10001
draft = false

[extra]
lang = "en"
translation = "docs/nettverk/dns.md"
+++

# Nameservers

We have two authoritative nameservers:

- ns1.fribyte.no - 158.37.6.21 (master)
- ns2.fribyte.no - 158.37.6.22 (slave)

In addition, we have fantonald, which is our recursive nameserver. It is used
by our servers and customers for whom we provide networking.

> During a transition period, we also have the old nameservers: kladden and
> svartepetter as authoritative servers, and kornelius as a recursive
> nameserver. These will be used until we have updated the nameservers at uib,
> srib, and bstv.
>
> Missing:
>
> - fribyte.uib.no
> - ss.uib.no
> - srib.no
> - bstv.no

## Setup for NS1 and NS2

NS1 and NS2 run bind9.

Zone files are located under `/var/bind/master` on NS1. This is where changes
to zones are made.

The slave servers fetch zone data using transfer requests. NS1 only allows
transfers to NS2 and ns.hyp.net. This should also include uib's nameserver at
some point.

Whenever NS1's zone files change, it sends a notify to the slave servers so
they can fetch the updated zones without delay.

NS1 and NS2 are authoritative nameservers. They do not perform recursive
lookups and only answer for their own zones.

### Transfer

When a zone changes on NS1, it sends a notify to all slaves. The slaves then
perform a transfer, meaning they fetch updated zone files from the master. The
IPs that are allowed to perform transfers are defined in `named.conf.options`.

### Configuration

Configuration can be found in `/etc/bind/named.conf.options` and
`/etc/bind/named.conf.local` on both NS1 and NS2. NS2 is configured as a
slave.

If you make a change to these files, always run

    $ named-checkconfig

No output indicates that the configuration is valid. After that, you can run

    $ sudo rndc reload

To modify zone files, see further down.

## Recursive DNS

Fantonald is our recursive nameserver, replacing kornelius. The goal is for all
machines on networks administered by friByte to use it.

Fantonald runs unbound, a simple recursive DNS server that requires little
configuration. It is configured to answer queries from the following networks:

- 158.37.6.0/25
- 10.250.0.0/16
- 2001:700:201:1::/64

## Reverse DNS

UiB will not further delegate reverse DNS for networks smaller than /24, so we
must send email to **hostmaster@uib.no** to make changes to the public part of
reverse DNS for our subnet.

We have one /64 IPv6 subnet for the server network, and reverse DNS for this
subnet has been delegated to us by Uninett. If you need to contact Uninett
about anything, use **drift@uninett.no**.

> TODO: fix reverse DNS for the new setup

## Making changes to zones

### What is a zone?

If you're not quite sure what a zone is, feel free to read this.
[https://ns1.com/resources/dns-zones-explained](https://ns1.com/resources/dns-zones-explained)

### Zone files in brief

A zone file consists of multiple resource records, which look roughly like
this:

```zone
<navn>  <ttl>   IN      <type>  <data>
; Eksempel:
wiki    1h      IN      A       158.37.6.4
```

If no name is given, the name from the line above will be used. If no TTL is
given, the value from the $TTL variable will be used.

Some common types:

| Type  | Use                                  |
| ----- | ------------------------------------ |
| A     | IPv4 address                         |
| AAAA  | IPv6 address                         |
| CNAME | Alias, points to another domain name |

#### Remember

- **When you add new domains to DNS, remember to set a new serial number so
  other DNS servers notice that it has been updated.**
  - The serial number must always be higher than the previous one.
- If you are only adding a subdomain, the files below probably already exist.

### Changing domains

To make changes to existing domains, log in to the master, which is NS1.

- make changes in the zone file (located in /var/bind/master)
- update **serial**
- Check that the zone file is correct:

```
$ named-checkzone <domene> /var/bind/master/<domene>
```

- Reload

```
$ sudo rndc reload
```

Remember to double-check with **dig** that the change has been applied; test on
both DNS servers

```sh
$ dig @158.37.6.21 <nyhost.domene.no>
$ dig @158.37.6.22 <nyhost.domene.no>
```

A helper script can also be used, which opens an editor for the zone, checks
validity, and reloads the zone.

```sh
$ ./update_zone.sh <domene>
```

### New domains

To add new domains to DNS, you must:

- Log in to the bind master server (ns1.fribyte.no)
- Create or edit the zone file /var/bind/master/<domene.no>
- Add the domain and contact person in /etc/bind/named.conf.local
- Reload the bind server
- Log in to the slave server (ns2.fribyte.no)
- Update /etc/bind/named.conf.local on the slave server with the new zone

**Zone file** (/var/bind/master/<domene.no>) should look roughly like this for
.no:

```zone
$ORIGIN fribyte.no.
$TTL 900
@       24h       IN      SOA     ns1.fribyte.no. hostmaster.fribyte.no (
                                2024220201      ; serial (YYMMDDSS)
                                1h              ; refresh
                                15m             ; retry
                                2w              ; expiry
                                2h              ; minimum
                        )

        24h     IN      NS      ns1.fribyte.no.
        24h     IN      NS      ns2.fribyte.no.
        24h     IN      NS      ns.hyp.net.
```

See [https://en.wikipedia.org/wiki/Zone_file wikipedia] for help with zone
files.

**named.conf.local** should look roughly like this on NS1:

```
zone "fribyte.no" {
        type primary;
        file "master/fribyte.no";
        // Hvis DNSSEC:
        dnssec-policy default;
        inline-signing yes;
};
```

**named.conf.local** should look roughly like this on NS2:

```
zone "fribyte.no" {
        type secondary;
        file "slave/fribyte.no";
        masters { NS1; };
        // Hvis DNSSEC:
        dnssec-policy default;
        inline-signing yes;
};
```

> Remember to run `rndc reload` to register new configs.

### Get rid of .jnl files

```sh
sudo rndc sync -clear
```

## Active domains

friByte has many domains. These are located in `/var/bind/master` on NS1.

### Global zones

| Domain name                              | Short explanation                                            |
| ---------------------------------------- | ------------------------------------------------------------ |
| 1.0.0.0.1.0.2.0.0.0.7.0.1.0.0.2.ip6.arpa | Reverse IPv6 zone for 2001:700:201:1::/64 (missing on the new setup) |
| fribyte.no                               | friByte - main domain                                        |
| fribyte.uib.no                           | friByte - being phased out                                   |
| ss.uib.no                                | Old student center - used on our machines - being phased out |
| bstv.no                                  | [Bergen Student TV](./kunder/bstv)                           |
| srib.no                                  | [Srib](./kunder/studentradioen)                              |
| rootlinjeforening.no                     | [Root Linjeforening](./kunder/root)                          |

### Local zones

| Domain name           | Short explanation              |
| --------------------- | ------------------------------ |
| 6.37.158.in-addr.arpa | Reverse zone for 158.37.6/24   |
| 250.10.in-addr.arpa   | Reverse zone for 10.250.0.0/16 |

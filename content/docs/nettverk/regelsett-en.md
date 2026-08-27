+++
title = "Production ruleset"
description = "A walkthrough of the pf ruleset that is in production"
template = "docs/page.html"
sort_by = "weight"
weight = 10001
draft = false

[extra]
lang = "en"
translation = "docs/nettverk/regelsett.md"
+++

### Ruleset walkthrough

A walkthrough of the ruleset we have in production. It may well be that this is
not exactly the same as the ruleset actually in production, but it should be
similar enough to explain how the ruleset is constructed.

### Macros

```sh
#       $OpenBSD: pf.conf,v 1.50 2011/04/28 00:19:42 mikeb Exp $
#
# See pf.conf(5) for syntax and examples.
# Remember to set net.inet.ip.forwarding=1 and/or net.inet6.ip6.forwarding=1
# in /etc/sysctl.conf if packets are to be forwarded between interfaces.


###########
# Macroes #
###########

# interfaces
ext_if = uplink
int_if = servers
pfsync_if = pfsync
kasse_if = kasse
klient_if = klient
mgmt_if = management


# subnets
uib_subnet = "129.177.0.0/16"
uninett_subnet = "158.37.0.0/16"
srib_subnet = "158.37.6.64/26"
mcast = "224.0.0.0/4"

# hosts and IPs
kasse_ip = "158.37.6.30"
klient_ip = "158.37.6.11"
tivoli_host = "128.177.30.195"
dole = "158.37.6.32"
doffen = "158.37.6.34"
bingen = "158.37.6.40"
guffen = "158.37.6.55"
klara = "158.37.6.58"
della = "158.37.6.48"
kornelius = "158.37.6.50"
happy = "158.37.6.57"
pluto = "158.37.6.59"
carp_uplink = "158.37.2.18"
garm = "158.37.6.3"


# Service Ports
tivoli_ports = "1500:1501"
tivoli_web_port = "1581"
mail_ports = "{ 25, 110, 143, 465, 587, 993, 995 }"
http_ports = "{ 80, 443 }"
jabber_ports = "{ 5222, 5223,5269, 9091 }"
munin_port = "4949"
ftp_ports = "20:21"
minecraft_port = "25565"
nethack_port = "23"
ntp_port = "123"

# host ports
guffen_ports = "{ 8080,8888, 8889, 4000, 4001 }"
guffen_ports_udp = "{ 4000:4001 }"


# icmp
icmp_types = "{ echoreq, unreach }"
```

The ruleset begins with useful macros (aka variables, referred to with `$` as in
bash/perl...). It is best to keep most interfaces, IPs, and ports as macros,
which makes the ruleset easier to maintain. If you want to see what the ruleset
looks like when all macros have been evaluated and expanded, you can run

```
# pfctl -nvf /etc/pf.conf
```

### Options, Limits and Timeouts

```sh
###########
# Options #
###########
set block-policy return
set loginterface egress
set skip on lo

set ruleset-optimization basic
set reassemble yes





##########
# Limits #
##########
set limit states 30000





############
# Timeouts #
############
set optimization normal
set timeout { tcp.established 3600, tcp.closing 60 }
```

Under options, a few important things happen. The default policy for blocking
traffic is set to return (the standard is drop); the reason we set this is that
it is easier to troubleshoot when you get feedback that traffic is not allowed
through.

Loginterface specifies which interface pf should collect statistics for. It is
common to set it to the **egress** interface group. The interface connected to
the **default gateway** is automatically added to the **egress** interface
group. The information that is collected can be seen by running

```sh
# pfctl -si
```

**set skip on lo** means that pf is not enabled for the **lo** interface group.
This group contains all loopback interfaces. The reason we include this option
is that it is very rarely useful to filter loopback traffic.

**set reassemble** ensures that fragmented packets are collected and put back
together before they are tested against the ruleset. This ensures that packets
cannot be sneaked through the firewall by using fragmentation tricks.

**set optimization normal** sets all timeouts for tcp and similar traffic to
"standard values".

**set timeout { tcp.established 3600, tcp.closing 60 }** lowers some of the tcp
timeout values because the standard values are very high.

### Tables, Queues and Normalization

```sh
##########
# Tables #
##########
# service tables
table <ssh> persist file "/etc/pf.d/tables/ssh"
table <dns> persist file "/etc/pf.d/tables/dns"
table <http> persist file "/etc/pf.d/tables/http"
table <mail> persist file "/etc/pf.d/tables/mail"

# block tables
table <martians> persist file "/etc/pf.d/tables/martians"
table <sshguard> persist
table <spamd> persist

# helper tables
table <firewall> const { $ext_if:0, $int_if:0, $pfsync_if:0, $kasse_if:0, $klient_if:0 }
table <localnets> const { $int_if:network, $kasse_if:network, $klient_if:network, $mgmt_if:network }
table <routablenets> const { $ext_if:network, $int_if:network }
table <authpf_users> persist
table <kvarteret_switches> const { 10.250.200.20, 10.250.200.21, 10.250.200.22 }




##########
# Queues #
##########
# No queues are configured yet!





#################
# Normalization #
#################
# No traffic normalization is configuret yet!
```

Here several tables are defined that we will use to make ingress filtering
simpler. The first tables are loaded from files under **/etc/pf.d/tables**;
these files are lists of IP addresses. The tables we define will later be used
in the ruleset to allow traffic for various services (ssh, http ...) to the
servers that are supposed to provide those services.

The middle tables (`martians`, `sshguard`, and `spamd`) are tables that contain
IPs and networks we want to block. **Martians** contains all IP addresses that
you should never receive traffic from on the internet, such as private IP
addresses, multicast addresses, and similar. The **sshguard** and **spamd**
tables are both application tables, which means that the contents of the tables
are controlled by "external" programs. `sshguard` is a program that scans the
SSH logs and adds addresses that are too aggressive to the **sshguard** table so
that we can drop traffic from those IPs. The **spamd** table is updated by the
`spamd` program, which downloads lists of known spammer IPs and puts them in the
**spamd** table so that none of those addresses can talk to our mail server.

The bottom tables are used to simplify rules where you want to match traffic
that should be routed to the internet and not internally. To do this, you list
all local networks in a table and then use the negation of that table in the
rule. For example:
`pass in on $int_if inet from $int_if:network to !<localnets> tag SERVER_OUT`

### Match (NAT)

```sh
#########
# Match #
#########
# NAT for Kvarteret -> Internet
match out on $ext_if inet from $kasse_if:network to !<localnets> received-on $kasse_if nat-to $kasse_ip scrub (no-df min-ttl 128 random-id reassemble tcp)
match out on $ext_if inet from $klient_if:network to !<localnets> received-on $klient_if nat-to $klient_ip scrub (no-df min-ttl 128 random-id reassemble tcp)

# NAT for Kvarteret -> Servers
match out on $int_if inet from $kasse_if:network to $int_if:network received-on $kasse_if nat-to $kasse_ip scrub (no-df min-ttl 128 random-id reassemble tcp)
match out on $int_if inet from $klient_if:network to $int_if:network received-on $klient_if nat-to $klient_ip scrub (no-df min-ttl 128 random-id reassemble tcp)
```

These are the rules that do NAT for Kvarteret's two networks (`klient` and
`kasse`). These rules only do NAT; they do not say anything about whether the
traffic is allowed through or not. That has to be handled with separate
**pass** rules in addition to these **match** rules.

**match out on $ext_if** means that this rule matches traffic that is going out
on the external interface. **inet from $kasse_if:network** means that the rule
matches IPv4 traffic from Kvarteret's kasse network. `to !<localnets>` means
that the rule only matches traffic going to the internet. **received-on
$kasse_if** checks that the traffic was received on the correct interface; this
prevents the rule from matching traffic from other interfaces. **nat-to
$kasse_ip** specifies which public IP the traffic should be mapped to.

NB!! All IPs that are used in a **nat-to $IP** clause must be configured on the
firewalls; they must be alias addresses on the carp interface that is connected
to the server network.

## Filters

### Standard block and traffic to firewall

```sh
###########
# Filters #
###########

# explicit blocks
block drop in log quick on $ext_if inet from urpf-failed
block drop in log quick on $ext_if inet from no-route
block drop in log quick on $ext_if inet from <martians>
block drop in log quick on $ext_if inet proto tcp from <sshguard> to self port 22


# default block
block log (to pflog1) set prio 0


# antispoof
antispoof quick for $ext_if
antispoof quick for $int_if
antispoof quick for $klient_if
antispoof quick for $kasse_if
antispoof quick for $mgmt_if





#######################
# traffic to firewall #
#######################
# the "keep state (no-sync) options is used here because traffic
# to to and from the firewalls them selves should not be synced
pass in quick inet proto tcp to self:0 port 22 keep state (no-sync) set prio (5, 6)
pass in quick inet proto icmp to self:0 keep state (no-sync) set prio 2
pass out quick inet from self:0 keep state (no-sync)
```

The firewall rules begin with rules that specifically block and drop unwanted
traffic.

**urpf-failed** means traffic that comes in on the wrong interface relative to
the IP it comes from, for example traffic from UiB arriving from Kvarteret's
network.

**no-route** explains itself: it matches traffic we do not have a route to. For
example, traffic from private IPs that the firewalls are not connected to.

`<martians>` is a table of all IPs you should never receive traffic from. All
traffic from the IPs listed in this table is dropped on its way in on the
external interface.

The last three rules ensure that we always have access to the firewalls via ssh
and icmp. These rules are written fairly generally so that they should match on
all interfaces. The last rule ensures that we can always send traffic from the
firewalls themselves.

The line that only contains **block** is what we call the standard block. This
rule ensures that all traffic not explicitly dropped or let through by other
rules is blocked with reset: tcp reset for tcp connections and icmp reset for
all other connections.

The **quick** keyword means that when traffic matches these rules, you do not
evaluate the rest of the ruleset, but stop the way you do in iptables.

**self:0** matches all addresses configured on the firewalls, except alias
addresses. If we had only used **self**, the rules would also have matched alias
addresses, which would have caused the firewalls to answer ssh and icmp on the
IP addresses that are used for Kvarteret's klient and kasse networks.

### External Interface

```sh
######################
# external interface #
######################


# external in TCP
#################
# the interface group called ingress is used here so that
# the rules can match incoming traffic from the internet,
# kvarteret kasse and kvarteret klient. This works because
# all three interfaces are a part of the ingress group.

# spamd
pass in quick on $ext_if inet proto tcp from <spamd> to $ext_if:network port smtp rdr-to 127.0.0.1 port 8025 set prio 0
pass in quick on $ext_if inet proto tcp to !<mail> port smtp rdr-to 127.0.0.1 port 8025 set prio 0

# authpf
anchor "authpf/*" in on $ext_if from <authpf_users>

# to servers
pass in on ingress inet proto tcp to <ssh> port 22 tag SERVER_IN set prio (5, 6)
pass in on ingress inet proto tcp to <mail> port $mail_ports tag SERVER_IN set prio (4, 6)
pass in on ingress inet proto tcp to <dns> port 53 tag SERVER_IN set prio (4, 6)
pass in on ingress inet proto tcp to <http> port $http_ports tag SERVER_IN set prio (4, 6)

# multicast (only for testing)
pass in on $ext_if inet from $mcast allow-opts tag IPTV


# ftp (kornelius)
pass in on $ext_if inet proto tcp to $kornelius port $ftp_ports tag SERVER_IN


# tivoli
pass in on $ext_if inet proto tcp from $uib_subnet to $int_if:network port $tivoli_ports tag SERVER_IN
pass in on $ext_if inet proto tcp from $uib_subnet to $int_if:network port $tivoli_web_port tag SERVER_IN


# samba (bingen)
pass in on $ext_if inet proto tcp from $srib_subnet to $bingen port 137:139 tag SERVER_IN
pass in on $ext_if inet proto tcp from $srib_subnet to $bingen port 445 tag SERVER_IN


# icecast (guffen)
pass in on $ext_if inet proto tcp to $guffen port $guffen_ports tag SERVER_IN


# jabber (klara)
pass in on $ext_if inet proto tcp to $klara port $jabber_ports tag SERVER_IN


# minecraft (happy)
pass in on $ext_if inet proto tcp to $happy port $minecraft_port tag SERVER_IN


# nethack (happy)
pass in on $ext_if inet proto tcp to $happy port $nethack_port tag SERVER_IN




# external in UDP
#################
# to servers
pass in on ingress inet proto udp to <dns> port 53 tag SERVER_IN set prio 4


# samba (bingen)
pass in on $ext_if inet proto udp from $srib_subnet to $bingen port 137:139 tag SERVER_IN
pass in on $ext_if inet proto udp from $srib_subnet to $bingen port 445 tag SERVER_IN


# icecast (guffen)
pass in on $ext_if inet proto udp to $guffen port $guffen_ports_udp tag SERVER_IN





# external in ICMP
##################
# to servers
pass in on ingress inet proto icmp to $int_if:network tag SERVER_IN set prio 2





# external out
##############

# egress filter
block drop out quick on $ext_if from ! <routablenets>  set prio 0
block drop out quick on $ext_if proto tcp from ! <mail> to port 25 set prio 0


# because we tag all outbound traffic we can just
# pass all the properly tagged traffic
pass out on $ext_if tagged SERVER_OUT set prio 4
pass out on $ext_if tagged NAT_INTERNET modulate state

```

### Internal Interface

```sh
######################
# internal interface #
######################

# servers to the internet
pass in on $int_if inet from $int_if:network to !<localnets> tag SERVER_OUT set prio 4


# munin to firewall
pass in on $int_if inet proto tcp from $della to port $munin_port keep state (no-sync)


# management (garm -> swtiches)
pass in on $int_if inet from $garm to <kvarteret_switches> tag MGMT



# internal out
##############
# because we tagged the traffic on the way
# in we can just let the tagged traffic pass
pass out on $int_if tagged SERVER_IN set prio 4
pass out on $int_if tagged NAT_SERVERS
pass out on $int_if tagged MGMT
```

### `kasse` Interface

```sh
###################
# kasse interface #
###################

# kasse in
##########
# kasse to internet
pass in on $kasse_if inet from $kasse_if:network to !<localnets> tag NAT_INTERNET


# kasse to servers
pass in on $kasse_if inet from $kasse_if:network to $int_if:network tag NAT_SERVERS
```

### `klient` Interface

```sh
####################
# klient interface #
####################

# klient in
###########
# klient to internet
pass in on $klient_if inet from $klient_if:network to !<localnets> tag NAT_INTERNET
pass in on $klient_if inet proto igmp from $klient_if:network to carp4


# klient to servers
pass in on $klient_if inet from $klient_if:network to $int_if:network tag NAT_SERVERS


# multicast IP TV
pass out on $klient_if inet to $mcast allow-opts tagged IPTV
```

### Management

```sh
###########
# mgmt_if #
###########

# in ntp
pass in on $mgmt_if inet proto udp to port $ntp_port

# in packetfence MAB
pass in on $mgmt_if inet from <kvarteret_switches> to $garm tag MGMT


# out
pass out on $mgmt_if tagged MGMT
```

### Carp

```sh
########
# carp #
########

# this ensures that carp traffic always gets through
pass inet proto carp set prio 7
```

### pfsync

```sh
####################
# pfsync interface #
####################

# allow the pfsync protocol
pass on $pfsync_if inet proto pfsync keep state (no-sync) set prio 7
```

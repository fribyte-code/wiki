+++
title = "Redundant firewall"
description = "Information about friByte's firewall"
template = "docs/page.html"
sort_by = "weight"
weight = 10001
draft = false

[extra]
lang = "en"
translation = "docs/nettverk/brannmur.md"
+++

**Contact person:** Mikal Sande <mikal.sande@gmail.com>

The redundant firewall project is finished, and this page is used to document the
setup. Dole and Doffen are the machines used as firewalls / routers.

### Redundancy and failover

Since our firewall setup consists of two redundant firewalls, it is useful to
know some commands for administering such a setup.

### Who is master?

The first thing you usually want to know is whether the firewall you have logged
into is master or slave.

```sh
# ifconfig carp
```

Shows all interfaces that are in the carp interface group. The firewall that is
master should have ''master'' status on all carp interfaces. Correspondingly,
the slave has ''slave'' status.

```sh
carp5: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:00:5e:00:01:05
        description: management, felles IP
        priority: 0
        carp: MASTER carpdev vlan200 vhid 5 advbase 1 advskew 0
        groups: carp
        status: master
        inet6 fe80::200:5eff:fe00:105%carp5 prefixlen 64 scopeid 0xe
        inet 10.250.200.1 netmask 0xffffff00 broadcast 10.250.200.255
```

### Change master

If you want to change the master, you can force a so-called ''graceful
failover'' by running

```sh
# ifconfig -g carp carpdemote 100
```

You set it back by running

```sh
# ifconfig -g carp -carpdemote 100
```

If you are going to shut down the master, you can either run the command above
or just turn off the machine. Carp is built into the operating system and will
ensure a proper failover when you shut it down.

If you restart the master, it will take over as master when it comes up again.
It will only do this if it has synced all the information it needs with the
slave (which was master while the master was down).

### Change master over SSH

If you are going to change the master over ssh, you must connect to the external
IPs on '''both firewalls'''. Connections to the internal addresses or the master
address will be broken when you change master.

```sh
ssh root@dole.uplink.ss.uib.no
ssh root@doffen.uplink.ss.uib.no
```

### Keep configs in sync

All config changes must be made on the master (`uplink.ss.uib.no`) firewall,
which under normal operation should be `dole.ss.uib.no`

Example of a config change on both firewalls

```sh
cd /etc/
vi dhcpd.conf
scp dhcpd.conf doffen:$PWD
```

### Keep the rule set in sync

The pfsync protocol ensures that all states are synced between the firewalls,
and carp ensures that the firewalls can share addresses and act as one
firewall. To keep the rule set itself in sync, we have a homemade script that
syncs things for us when we have made changes.

### Procedure

```sh
vi /etc/pf.conf
sh /root/syncpf.sh
```

### Add or remove IPs from the ssh table

```sh
vi /etc/pf.d/tables/ssh
sh /root/syncpf.sh
```

The sync script checks that the rule set is valid and will not sync a rule set
that is invalid. The sync script copies **/etc/pf.conf** and **/etc/pf.d/** to
the other firewall and then loads the new rule set on both firewalls.

### Firewall / Router Setup

What follows is documentation and an explanation of what runs on the firewalls,
given in order of priority based on how important it is for friByte's network.

### Network setup

First, you set up all the network cards on the firewalls, first the physical
network cards and then the logical ones. All network card settings in OpenBSD
are placed in separate files called hostname.if (where if is the name of the
interface) under /etc/.

### Physical network cards

/etc/hostname.em2 (uplink to UiB)

```sh
inet 158.37.2.19 255.255.255.248
dest 158.37.2.17
group uplink
group ingress
description "Uplink til UiB"
```

/etc/hostname.em1 (server network)

```sh
inet 158.37.6.32 255.255.255.192
group servers
description "server nettverk"
```

/etc/hostname.em0 (vlan trunk for client, cash register, and management)

```sh
up
description "vlan trunk"
```

/etc/hostname.bge0 (sync interface for pfsync and dhcp-sync)

```sh
inet 10.7.7.1 255.255.255.0
group pfsync
description "pfsync interface"
```

Some information

- **em2** is our uplink towards UiB, and since the connection to the firewalls'
  default gateway is located there, we add **dest 158.37.2.17**. This IP is
  also placed in the file **/etc/mygate**.
- **em1** is the network card connected to the server network.
- **em0** does not have an IP because that interface is used as a VLAN trunk
  for various networks that we will come back to.
- **bge0** is set up with a private IP address (RFC1918) because it is the
  interface used to sync the state table between the firewalls. This syncing is
  what allows the firewalls to take over for each other without any network
  sessions being interrupted.

### Logical network cards

The logical network cards are set up in the same way as the physical ones, with
one small difference. The name given to a logical network card determines the
function, and you can give a logical network card whichever number you want.
This is something we have taken advantage of to make it easier to remember the
associations between the logical ones and the networks they belong to.

### VLAN

/etc/hostname.vlan1

```sh
inet 10.250.1.2 255.255.255.0 10.250.1.255 vlan 101 vlandev em0
group kasse
group ingress
description "kasse nettverk"
```

/etc/hostname.vlan10

```sh
inet 10.250.10.2 255.255.254.0 10.250.11.255 vlan 110 vlandev em0
group klient
group ingress
description "klient nettverk"
```

/etc/hostname.vlan200

```sh
inet 10.250.200.2 255.255.255.0 10.250.200.255 vlan 200 vlandev em0
group management
description "switch management-nett"
```

We have three VLANs that come into the firewalls via **em0**. One is a
management network that should not have any internet access. The other two are
networks that we are going to run NAT (Network Address Translation) on for
Kvarteret.

### CARP

Now that all the physical and logical networks are set up, it is time to set up
the logical interfaces for the IP addresses the firewalls will share. This is
where CARP comes in. When you use CARP, you need at least three IPs for the
routers on all the networks they are going to be part of. They each need their
own IP and one shared IP. When everything is up and running, only the master
firewall should use the shared IP.

/etc/hostname.carp1 (server network)

```sh
vhid 1 pass derpderpderp carpdev em1 advskew 0 158.37.6.33 netmask 255.255.255.192
alias 158.37.6.11 netmask 255.255.255.192
alias 158.37.6.30 netmask 255.255.255.192
description "server nett, felles IP"
```

/etc/hostname.carp2 (uplink to UiB)

```sh
vhid 2 pass roflrofllolroflrofl carpdev em2 advskew 0 158.37.2.18 netmask 255.255.255.248
description "uplink, felles IP"
```

/etc/hostname.carp3 (cash register network)

```sh
vhid 3 pass roflmao carpdev vlan1 advskew 0 10.250.1.1 netmask 255.255.255.0
description "kasse, felles IP"
```

/etc/hostname.carp4 (client network)

```sh
vhid 4 pass w00t carpdev vlan10 advskew 0 10.250.10.1 netmask 255.255.254.0
description "klient, felles IP"
```

/etc/hostname.carp5 (management network)

```sh
vhid 5 pass  carpdev vlan200 advskew 0 10.250.200.1 netmask 255.255.255.0
description "management, felles IP"
```

**carp1** stands out a little here because it has some alias addresses, because
the firewalls must themselves own the addresses that are used for NAT for the
cash register and client networks, otherwise problems can arise.

The rest of the CARP interface setup is the same. **vhid** is just the ID of
the interface.

- **pass** is the password used to encrypt and authenticate all CARP packets
  between the firewalls for the relevant interface. The passwords have been
  replaced with "funny stuff" because it is bad style to have passwords lying
  around on the wiki :)
- **carpdev** is the name of the interface this carp interface should use.
- **advskew** says something about how often the firewall announces that it is
  up. The firewall that announces itself most often becomes master, so it is
  important that the firewalls are set up with different **advskew**.
- And the rest of the setup with IPs and netmasks must match the settings on
  the interface that is **carpdev**.

IMPORTANT!! **avdskew** must be set differently on the firewalls. The one you
want to be master should have **advskew** set to **0** for all its CARP
interfaces. On the other firewall, you can set **advskew** to **100**.

ALSO IMPORTANT!! The passwords used on the different carp interfaces should be
different and hard to crack. Since this password is only shared between the
firewalls, you can create a strong password by running

```sh
openssl rand -base64 40
```

### sysctl

When the network cards are set up, there are a couple of sysctl settings we
must change so that the firewalls can "forward" traffic. First we make the
change with the ''sysctl'' command, then we add it to ''/etc/sysctl.conf'' to
make the change permanent.

#### To be able to forward IPv4 traffic

```sh
# sysctl net.inet.ip.forwarding=1                                                                                    net.inet.ip.forwarding: 0 -> 1
```

/etc/sysctl.conf (remove the comment from the line)

```sh
net.inet.ip.forwarding=1
```

#### To be able to forward IPv6 traffic

```sh
# sysctl net.inet6.ip6.forwarding=1
net.inet6.ip6.forwarding: 0 -> 1
```

/etc/sysctl.conf (remove the comment from the line)

```sh
net.inet6.ip6.forwarding=1
```

#### So that carp works the way we want

```sh
# sysctl net.inet.carp.preempt=1                                                                                     net.inet.carp.preempt: 0 -> 1
```

/etc/sysctl.conf (remove the comment from the line)

```sh
net.inet.carp.preempt=1
```

#### Because we have four physical network cards

```sh
# sysctl net.inet.ip.ifq.maxlen=1024                                                                                 net.inet.ip.ifq.maxlen: 256 -> 1024
```

/etc/sysctl.conf (add the line)

```sh
net.inet.ip.ifq.maxlen=1024
```

### pf

pf is the firewall that comes with OpenBSD; the name is an abbreviation for
''packet filter''. For the firewalls to start routing traffic, you do not need
any large rule set. You can start with a very simple one and then add things
gradually.

#### Minimal rule set

/etc/pf.conf

```sh
pass
```

This makes the firewalls behave like routers. The server network will work
fine, but not the cash register and client networks.

#### Minimal rule set with NAT

/etc/pf.conf

```sh
# interfaces
ext_if = uplink
int_if = servers
kasse_if = kasse
klient_if = klient

# hosts and IPs
kasse_ip = "158.37.6.30"
klient_ip = "158.37.6.11"


# kasse nett
match out on $ext_if inet from $kasse_if:network nat-to $kasse_ip
match out on $ext_if inet from $klient_if:network nat-to $klient_ip

# klient nett
match out on $int_if inet from $kasse_if:network to $int_if:network nat-to $kasse_ip
match out on $int_if inet from $klient_if:network to $int_if:network nat-to $klient_ip

pass
```

With this rule set, we will have a working network without any firewall
filtering.

### Rule set

To provide good insight into how the pf setup on the firewalls works, separate
pages have been created for how to write rule sets and a review of the rule set
that is in use.

[Review of the rule set in production](/docs/nettverk/regelsett)

[How to write a rule set](/docs/nettverk/regelsett-tutorial)

### Ideas used to simplify the rule set

There are several ways to write rule sets in pf; the way our rule set is
written is intended to make it understandable and easy to change. To do this,
we use several ideas / techniques that simplify the work.

### block by default

All incoming traffic is blocked and reset unless there is a rule that explicitly
allows the traffic. Of outgoing traffic (to the Internet), we allow most of it,
with a couple of exceptions.

Blocks all traffic that is not explicitly allowed.

```sh
block log (to pflog1) set prio 0
```

### pf tables

To make filtering incoming traffic easier, we use ''tables''. These are usually
stored in files under ''/etc/pf.d/tables'' and are only lists of IPs. We have
lists for the different services we have available, so there is a list for
''http'', ''ssh'', etc. So if a server is going to run a service that should be
available via the Internet, you only have to add the server's IP to the correct
list and then reload the firewall rules.

Lets traffic from the Internet through to our web servers.

```sh
table <http> persist file "/etc/pf.d/tables/http"
pass in log on ingress inet proto tcp to <http> port $http_ports tag SERVER_IN set prio (4, 6)
pass out on $int_if tagged SERVER_IN modulate state (pflow) set prio 4
```

### Interface groups

Our network has several ingresses and only one egress (uplink to UiB), which
means that we often have to allow traffic into our servers from several
networks, for example from the Internet, Kvarteret's client network, and
Kvarteret's cash register network. To avoid having to duplicate several firewall
rules to let in the same traffic from these networks, we use **interface
groups**.

With interface groups, we can put several network cards into the same group,
for example the **ingress** group. We can do this with all interfaces that need
to send traffic to our servers (Internet, Kvarteret client, etc...) and in that
way we only need to write the rules for what we want to let in to our servers
once. This saves us a lot of rule duplication and unnecessary rule-set
maintenance.

Lets traffic through to the Minecraft server from all our ingress networks.
(Internet, client, cash register, and public)

```sh
happy = "158.37.6.57"
minecraft_port = "25565"
pass in log on ingress inet proto tcp to $happy port $minecraft_port tag SERVER_IN
pass out on $int_if tagged SERVER_IN modulate state (pflow) set prio 4
```

### tagging

When traffic passes through a router, the traffic has to go through at least two
network cards. For a firewall, that means all traffic must be evaluated against
the rule set twice before it is let through. This can quickly lead to a lot of
unnecessary duplication of rules. So instead of maintaining fairly similar rules
in several places in the rule set, we can use **tags**. You tag traffic when it
comes into the firewall and use the tag to let the traffic through on its way
out of the firewall.

Lets web, mail, dns, and ssh through from all ingress networks (Internet,
client, cash register, and public) to our servers.

```sh
mail_ports = "{ 25, 110, 143, 465, 587, 993, 995 }"
http_ports = "{ 80, 443 }"

table <ssh> persist file "/etc/pf.d/tables/ssh"
table <dns> persist file "/etc/pf.d/tables/dns"
table <http> persist file "/etc/pf.d/tables/http"
table <mail> persist file "/etc/pf.d/tables/mail"

pass in log on ingress inet proto tcp to <ssh> port 22 keep state (max-src-conn 20, max-src-conn-rate 10/10, overload <sshbrutes> flush) tag SERVER_IN set prio (5, 6)
pass in log on ingress inet proto tcp to <mail> port $mail_ports tag SERVER_IN set prio (4, 6)
pass in log on ingress inet proto tcp to <dns> port 53 tag SERVER_IN set prio (4, 6)
pass in log on ingress inet proto tcp to <http> port $http_ports tag SERVER_IN set prio (4, 6)

pass out on $int_if tagged SERVER_IN modulate state (pflow) set prio 4
```

### spamd

Spamd is part of the standard installation; you do not need to install any
packages.

#### /etc/rc.conf.local on dole

```sh
spamd_flags="-Y bge0 -y bge0"
spamlogd_flags="-Y 10.7.7.2"
```

#### /etc/rc.conf.local on doffen

```sh
spamd_flags="-v -Y bge0 -y bge0"
spamlogd_flags="-Y 10.7.7.1
```

#### /etc/pf.conf

```sh
# spamd
table <nospamd> persist file "/etc/pf.d/tables/nospamd"
table <spamd-white> persist
table <spamd> persist

# spamd
pass in quick on $ext_if inet proto tcp from <spamd> to port smtp rdr-to 127.0.0.1 port 8025 set prio 0
pass in quick log on $ext_if inet proto tcp from <nospamd> to <mail> tag SERVER_IN
pass in quick log on $ext_if inet proto tcp from <spamd-white> to <mail> tag SERVER_IN
pass in quick on $ext_if inet proto tcp to port smtp rdr-to 127.0.0.1 port 8025 set prio

# sync
pass on $pfsync_if inet proto udp to port 8025 keep state (no-sync) set prio 6
```

#### /etc/mail/spamd.conf

```sh
all:\
        :uatraps:nixspam:bsdly:openbl:drop:edrop:local:

# University of Alberta greytrap hits.
# Addresses stay in it for 24 hours from time they misbehave.
uatraps:\
        :black:\
        :msg="Your address %A has sent mail to a ualberta.ca spamtrap\n\
        within the last 24 hours":\
        :method=http:\
        :file=www.openbsd.org/spamd/traplist.gz

# Nixspam recent sources list.
# Mirrored from http://www.heise.de/ix/nixspam
nixspam:\
        :black:\
        :msg="Your address %A is in the nixspam list\n\
        See http://www.heise.de/ix/nixspam/dnsbl_en/ for details":\
        :method=http:\
        :file=www.openbsd.org/spamd/nixspam.gz

# bsdly.net greytrapping blacklist
bsdly:\
        :black:\
        :msg="Your address %A is in the bsdly list.":\
        :method=http:\
        :file=www.bsdly.net/~peter/bsdly.net.traplist

# openbl.org
openbl:\
        :black:\
        :msg="Your address %A is in the openbl.org list.":\
        :method=http:\
        :file=www.openbl.org/lists/base.txt

# Spamhaus drop
drop:\
        :black:\
        :msg="Your address %A is in the spamhaus.org drop list.":\
        :method=http:\
        :file=www.spamhaus.org/drop/drop.txt

# Spamhaus edrop
edrop:\
        :black:\
        :msg="Your address %A is in the spamhaus.org edrop list.":\
        :method=http:\
        :file=www.spamhaus.org/drop/edrop.txt

# Local blacklist
local:\
        :black:\
        :msg="Your address %A is our blacklist. Please stop sending us spam.":\
        :method=file:\
        :file=/etc/mail/blacklist.txt
```

#### Start spamd for the first time

```sh
pfctl -f /etc/pf.conf
/etc/rc.d/spamlogd start
/etc/rc.d/spamd start
/usr/libexec/spamd-setup
```

### Whitelisting with SPF and MX

To whitelist certain domains, we use a couple of scripts to parse SPF records
for the domains. The scripts extract IPs and place them in a table in pf so
that we can let mail from the domain in without going through greylisting. When
we then receive mail from these domains, they will be whitelisted by spamd, so
this is mostly a bootstrap technique.

MX records for the domain are also added to the whitelist.

The list of domains that are whitelisted is located in
**/root/spamd_whitelist.sh**.

The scripts

```sh
/root/spamd_whitelist.sh
/root/spf_dump.py
```

Crontab

```sh
10      1       *       *       *       /bin/sh /root/spamd_whitelist.sh
```

### Incoming whitelisting

**-I** makes spamlogd whitelist addresses only based on incoming smtp traffic.
It can be a bit problematic to whitelist outgoing smtp traffic because of
bounces and similar things.

```sh
spamlogd_flags="-Y 10.7.7.2 -I"
```

### greyscanner

Greyscann is a daemon written in perl that every 10 minutes checks all
greylisted entires in the **spamdb** database. If it finds some entries that do
not have an A or MX record in DNS, they are **trapped** for a day. All IPs that
are **TRAPPED** are only allowed to talk to **spamd** instead of our mail
servers.

Install

```sh
# pkg_add -vi greyscanner
```

#### /etc/rc.conf.local

```sh
pkg_scripts="greyscanner"
```

### graytrapping

To annoy spammers even more,
[greytrapping](http://home.nuug.no/~peter/pf/en/spamd.greytrapping.html) is set
up on the firewalls. The idea is that we get spammers to scrape several
fictitious mail addresses from our website (fribyte.uib.no/mail/mail.txt), and
when they then try to send mail to these addresses (which should never receive
legitimate mail), we can safely blacklist them.

### Local blacklist

We have a local blacklist we can add IPs to. There are some organizations that
make a living by sending advertisements for others; organizations like that can
just as well be blocked completely.

#### Add IPs to the blacklist

Add the IP you want to blacklist to ''/etc/mail/blacklist.txt

#### Remove blacklisted IPs from the spamd database

```sh
sh /root/spamd-remove-blacklisted.sh
```

#### Restart spamd

```sh
/etc/rc.d/spamd restart
```

**Remember to make the changes on both firewalls!**

```sh
cd /etc/mail
scp blacklist.txt doffen:$PWD
ssh doffen
  sh /root/spamd-remove-blacklisted.sh
  /etc/rc.d/spamd restart
```

## Notes

### dhcpd

dhcpd is part of the standard installation; nothing to install.

#### /etc/dhcpd.conf

```sh
authoritative;

default-lease-time 3600;

shared-network KASSE-NET {
        option domain-name "kasse.kvarteret.no";
        option domain-name-servers 158.37.6.53, 158.37.6.52;

        subnet 10.250.1.0 netmask 255.255.255.0 {
                option routers 10.250.1.1;
                range 10.250.1.100 10.250.1.200;
        }
}

shared-network SERVER-NET {
        option domain-name "ss.uib.no";
        option domain-name-servers 158.37.6.52, 158.37.6.53;

        allow booting;
        allow bootp;

        use-host-decl-names on;

        subnet 158.37.6.0 netmask 255.255.255.192 {
                option routers 158.37.6.33;
                filename "pxelinux.0";
                next-server 158.37.6.40;

                host bestemor {
                        hardware ethernet 00:1d:09:01:cb:68;
                        fixed-address 158.37.6.35;
                }

                host bestefar {
                        hardware ethernet 00:25:90:05:ca:0e;
                        fixed-address 158.37.6.37;
                }

                host hetti {
                        hardware ethernet 00:1e:4f:17:2a:d0;
                        fixed-address 158.37.6.36;
                }

                host netti {
                        hardware ethernet 00:1e:c9:d4:d8:b2;
                        fixed-address 158.37.6.44;
                }

                host letti {
                        hardware ethernet 00:1e:4f:17:13:e8;
                        fixed-address 158.37.6.45;
                }

                host della {
                        hardware ethernet 00:19:b9:de:19:44;
                        fixed-address 158.37.6.48;
                }

                host ole {
                        hardware ethernet 00:19:b9:de:20:0d;
                        fixed-address 158.37.6.42;
                }

                host hermes {
                        hardware ethernet 00:30:48:34:a2:f4;
                        fixed-address 158.37.6.14;
                }

                host draugen {
                        hardware ethernet 00:50:04:41:30:5c;
                        fixed-address 158.37.6.17;
                }
        }
}
```

#### /etc/rc.conf.local

```sh
dhcpd_flags="-Y bge0 -y bge0 vlan1 em1"
```

### ntpd ==

ntpd is part of the standard installation.

#### /etc/ntpd.conf

```sh
# Bruk ntp.uib.no som atorativ kilde
server ntp.uib.no
server ntp.uio.no
#server time.hint.no
server ntp.uninett.no

# Synkroniser med andre maskiner
server bestefar.ss.uib.no
server bestemor.ss.uib.no
server hetti.ss.uib.no
server netti.ss.uib.no
server letti.ss.uib.no

# Local users may interrogate the ntp server more closely.
listen on 127.0.0.1
listen on ::1

listen on 158.37.6.32
listen on 158.37.6.33
listen on 10.250.200.1
listen on 10.250.200.2
```

#### /etc/rc.conf.local

```sh
ntpd_flags="-s"
```

The `-s` flag ensures that the clock is roughly adjusted at startup if the
clock is minutes or more off.

### munin-node

#### install

```sh
# pkg_add -vvi munin-node
```

then you configure it however you like...

#### /etc/rc.conf.local

```sh
pkg_scripts="munin_node"
```

### sshguard

#### install

```sh
# pkg_add -vvi sshguard
```

#### /etc/pf.conf

```sh
table <sshguard> persist
block drop in log (to pflog1) quick on $ext_if inet proto tcp from <sshguard> to self port 22 set prio 0
```

#### /etc/rc.conf.local

```sh
sshguard_flags=""
pkg_scripts="sshguard"
```

## Backup

We do not have direct backups on the OpenBSD firewalls. Instead, we have a
script **/root/config-backup.sh** that copies all important config files over to
the other firewall and to bingen, which has backup towards UiB. The reason for
this is that it is easier to reinstall OpenBSD and copy over the old config
files than to restore a complete backup from backup. Both firewalls copy files
into their own directory under **/home/config-backup** on bingen and under
**/root** on the other firewall.

Reinstallation is done by reinstalling OpenBSD and then copying over all the
config files from bingen. When the reinstalled firewall comes up again, it
should be ready to route traffic.

## Links

Howtos:

- [Presentation of PF](http://home.nuug.no/~peter/pf/eurobsdcon2012/index.html)
- [Introduction to PF](http://home.nuug.no/~peter/pf/en/)
- [Calomel Howto PF](https://calomel.org/pf_config.html)
- [Rule set optimization](http://undeadly.org/cgi?action=article&sid=20060927091645)

### Important manuals

- [Manual: pf.conf](http://www.openbsd.org/cgi-bin/man.cgi?query=pf.conf&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html)
- [Manual: pfctl](http://www.openbsd.org/cgi-bin/man.cgi?query=pfctl&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html)

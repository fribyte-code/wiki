+++
title = "How to write pf rulesets"
description = "An explanation of how to write a ruleset in pf"
template = "docs/page.html"
sort_by = "weight"
weight = 10001
draft = false

[extra]
lang = "en"
translation = "docs/nettverk/regelsett-tutorial.md"
+++

A brief procedure for writing pf rulesets; other howtos and some prior knowledge
of pf are useful to have beforehand. The first part should be considered the
minimum amount of work that must be done to get the firewalls working. If there
is ever a crisis and we have to reinstall or set up the machines again, the
first part is the most important. The rest is "nice to have" and is not
necessary for the network to function.

The two most important functions the machine must perform are:

- Routing
- Firewall

If there are problems and something needs to be fixed, these functions should
have priority.

### Get things working

First we want to get things working: let our servers reach the outside world and
let the internet reach us on the ports and services we want to give it access
to.

### Set up sections for the ruleset

These are the sections we divide the ruleset into. This is something pf expects,
and it keeps everything structured.

```sh
###########
# Macroes #
###########


##########
# Tables #
##########


###########
# Options #
###########


##########
# Queues #
##########


#################
# Normalization #
#################


#########
# Match #
#########


##################
# Packet filters #
##################
```

### First, a bit of prep work

We start by filling in information about network cards, IPs, and similar
details. It is useful to have when we are going to write the filter rules. The
options that are set are standard: we set which interface we want to log
statistics for and what we should do when some traffic is blocked. We also set
up our first filter rule, which blocks everything by default. When we later add
more rules, they will only allow what they specify and everything else will be
blocked.

```sh
###########
# Macroes #
###########
ext_it = "em0"
int_if = "em1"
nat_if = "bge0"
nat_ip = "158.37.6.11"


##########
# Tables #
##########


###########
# Options #
###########
set block-policy return
set loginterface $ext_if


##########
# Queues #
##########


#################
# Normalization #
#################


#########
# Match #
#########


##################
# Packet filters #
##################
block
```

### Let traffic through from the outside

To get any use out of the firewall, it has to let traffic through. The first
thing we want to do is create rules that allow traffic through from the internet
to the services we run. We do this per IP, so to keep the ruleset from becoming
too large we use tables for each service. Tables also make the administration of
the ruleset easier. Each table contains IPs, and each list contains the IPs of
the servers that run the relevant service.

```sh
##########
# Tables #
##########
table <ssh> persist file "/etc/pf.d/tables/ssh"
table <dns> persist file "/etc/pf.d/tabes/dns"
table <http> persist file "/etc/pf.d/tables/http"
table <mail> persist file "/etc/pf.d/tables/mail"

###########
# Filters #
###########
pass in on $ext_if inet proto tcp to <ssh> port 22
pass in on $ext_if inet proto tcp to <mail> port 25
pass in on $ext_if inet proto tcp to <http> port 80,443
pass in on $ext_if inet proto udp to <dns> port 53

pass out on $int_if inet proto tcp to <ssh> port 22
pass out on $int_if inet proto tcp to <mail> port 25
pass out on $int_if inet proto tcp to <http> port 80,443
pass out on $int_if inet proto udp to <dns> port 53
```

These rules allow traffic through from the internet to the services we run, but
there is a fair amount of repetition in the rules for the external and internal
interface. We can simplify this a bit by using tags. Tags in pf are only an
internal marker that pf uses, and you can filter using tags.

```sh
##########
# Tables #
##########
table <ssh> persist file "/etc/pf.d/tables/ssh"
table <dns> persist file "/etc/pf.d/tabes/dns"
table <http> persist file "/etc/pf.d/tables/http"
table <mail> persist file "/etc/pf.d/tables/mail"


###########
# Filters #
###########
pass in on $ext_if inet proto tcp to <ssh> port 22 tag EXT_IN
pass in on $ext_if inet proto tcp to <mail> port 25 tag EXT_IN
pass in on $ext_if inet proto tcp to <http> port 80,443 tag EXT_IN
pass in on $ext_if inet proto udp to <dns> port 53 tag EXT_IN

pass out on $int_if tagged EXT_IN
```

With the help of tags, we have simplified the ruleset a bit and made its
administration somewhat easier; just remember to tag new rules when they are
added.

### Let traffic through from the inside

To let traffic out from the inside, we do the same as we did above. The
difference is that we tag on the internal interface and allow it out by using a
tag on the external interface.

```sh
###########
# Filters #
###########
# external in
pass in on $ext_if inet proto tcp to <ssh> port 22 tag EXT_IN
pass in on $ext_if inet proto tcp to <mail> port 25 tag EXT_IN
pass in on $ext_if inet proto tcp to <http> port 80,443 tag EXT_IN
pass in on $ext_if inet proto udp to <dns> port 53 tag EXT_IN

# external out
pass out on $ext_if tagged INT_OUT


# internal in
pass in on $int_if inet from $int_if:network tag INT_OUT

# internal out
pass out on $int_if tagged EXT_IN
```

### The order of the rules

To make the ruleset structured and efficient, it is a good idea to write the
rules in a certain order. The first step is to have separate sections for each
interface. That makes it easier to find your way around the ruleset and matches
better with how pf evaluates the ruleset. The rules above are arranged by
interface, direction, and protocol.

pf evaluates rules in this order: (shamelessly taken from
[here](http://www.undeadly.org/cgi?action=article&sid=20060927091645))

```sh
  1. interface ('on fxp0')
  2. direction ('in', 'out')
  3. address family ('inet' or 'inet6')
  4. protocol ('proto tcp')
  5. source address ('from 10.1.2.3')
  6. source port ('from port < 1024')
  7. destination address ('to 10.2.3.4')
  8. destination port ('to port 80')
```

Because pf evaluates rules in this order, it is a good idea to write the
ruleset in the same order :) In most cases it is enough to follow the first
four points; there is not as much to gain from the last four, so they can often
be ignored.

### Where should you filter what?

Writing rulesets for a firewall that is also a router is somewhat different from
writing rulesets for a host that should have its own firewall. One thing that
can cause confusion is that a `pass` rule does not guarantee that packets get
through. You need rules for both the incoming interface and the outgoing
interface for traffic to get through. We have already fixed this by using tags
to reduce the work, but where should different things be filtered?

One way to do it is to do all filtering on one interface. The downside is that
some traffic may be allowed in on one interface but get blocked when it is about
to go out. That is not really a big danger, but it can be useful to block
traffic as early as possible.

Another way to do it is to always filter on the incoming interface and then use
tags to let the traffic out on the outgoing interface. This is how our rules
have been written so far. _Honestly, this is my (Mikal's) personal preference;
others may disagree._

### NAT

If we are going to give internet access to a client network (such as
Kvarteret), we need NAT. Fortunately, NAT is very simple in pf.

```sh
###########
# Matches #
###########
match out on $ext_if inet from $nat_if:network received-on $nat_if nat-to $nat_ip tag NAT


###########
# Filters #
###########
pass out on $ext_if inet tagged NAT

pass in on $nat_if from $nat_if:network
```

The filter rules are already familiar; the only new thing is the match rule.
Match rules can mostly take the same arguments as `pass` and `block` rules. The
difference is that match rules neither block nor allow traffic through. What
they do is perform the specified action on all packets that match the rule,
which in this case is address translation in the form of **nat-to $nat_ip**.

### Rules for the firewall itself

We now have traffic flowing in both directions, as well as NAT for a client
network. Now it is time to differentiate between traffic that should go through
the firewall vs. traffic that should go to the firewall. For the external
interface, this is fairly easy since we have already used tables to allow
traffic in. In the first round, we settle for only allowing ssh and ping into
the firewall. Here we use the `self` keyword. And since we want the firewall to
always be available via ssh and ping, we simplify the rule so that it applies to
all interfaces. We also want all traffic from the firewall to be allowed
through.

```sh
pass in quick inet proto tcp to self:0 port 22
pass in quick inet proto icmp to self:0
pass out quick inet from self:0
```

When it comes to the server and client network, we need to do a bit more. We
create a table, `<localnets>`, containing the local networks the router is
connected to, and use this table to differentiate between traffic with local vs.
global endpoints. The commented-out rules are the old version of the rules.

```sh
# internal in
#pass in on $int_if inet from $int_if:network tag INT_OUT
pass in on $int_if inet from $int_if:network to ! <localnets> tag INT_OUT

# nat in
#pass in on $nat_if inet from $nat_if:network
pass in on $nat_if inet from $nat_if:network to ! <localnets>
```

### The ruleset so far

```sh
###########
# Macroes #
###########
ext_it = "em0"
int_if = "em1"
nat_if = "bge0"
nat_ip = "158.37.6.11"


##########
# Tables #
##########
table <ssh> persist file "/etc/pf.d/tables/ssh"
table <dns> persist file "/etc/pf.d/tabes/dns"
table <http> persist file "/etc/pf.d/tables/http"
table <mail> persist file "/etc/pf.d/tables/mail"


###########
# Options #
###########
set block-policy return
set loginterface $ext_if


##########
# Queues #
##########


#################
# Normalization #
#################


#########
# Match #
#########
match out on $ext_if inet from $nat_if:network received-on $nat_if nat-to $nat_ip tag NAT


##################
# Packet filters #
##################
# default block
block

# traffic to and from the firewall
pass in quick inet proto tcp to self:0 port 22
pass in quick inet proto icmp to self:0
pass out quick inet from self:0

# external in
pass in on $ext_if inet proto tcp to <ssh> port 22 tag EXT_IN
pass in on $ext_if inet proto tcp to <mail> port 25 tag EXT_IN
pass in on $ext_if inet proto tcp to <http> port 80,443 tag EXT_IN
pass in on $ext_if inet proto udp to <dns> port 53 tag EXT_IN

# external out
pass out on $ext_if inet tagged INT_OUT
pass out on $ext_if inet tagged NAT


# internal in
pass in on $int_if inet from $int_if:network to ! <localnets> tag INT_OUT
pass in on $int_if inet proto tcp from $int_if:network to $int_if port 22
pass in on $int_if inet proto icmp icmp-type echoreq from $int_if:network to $int_if

# internal out
pass out on $int_if inet tagged EXT_IN


# nat in
pass in on $nat_if inet from $nat_if:network to ! <localnets>
pass in on $nat_if inet proto tcp from $nat_if:network to $int_if port 22
pass in on $nat_if inet proto icmp icmp-type echoreq from $nat_if:network to $nat_if
```

### Adding a bit more functionality

Once the configuration in the section above is complete, you should have a
working ruleset that allows traffic to flow in both directions. This section
covers things that can easily be added after you have pf up and running.

### Block a few more things

In addition to having a block-by-default policy, it is a good idea to add some
explicit blocks for things we can be sure are not valid traffic. All traffic
that matches these block rules should be dropped; there is no reason to respond
to invalid traffic.

A short list

- [Martian packets, kept in a separate table](https://en.wikipedia.org/wiki/Martian_packet)
- Connections that violate
  [urpf](https://en.wikipedia.org/wiki/Reverse_path_forwarding#Unicast_RPF_.28uRPF.29)
- IPs we have no route to
- Connections that are clearly spoofed (using the `antispoof` keyword)

Unlike the rules we have written so far, these rules should use the `quick`
keyword. `quick` means that packets matching the rule do not continue being
evaluated against the rest of the ruleset; `quick` makes rules behave like in
ipfilter / iptables.

```sh
antispoof quick for $ext_if

block drop in quick on $ext_if inet from urpf-failed set prio 0
block drop in quick on $ext_if inet from no-route set prio 0
block drop in quick on $ext_if inet from <martians> set prio 0
```

Antispoof is just a shortcut; the one rule is expanded into several rules. For
example, if `$ext_if` has the IP `158.37.6.4`, `antispoof quick for $ext_if`
expands to

```sh
block quick drop in on ! $ext_if inet from 158.37.6.4/32 to any
block quick drop in inet from 158.37.6.4 to any
```

### Egress filters

It can be difficult and somewhat unnecessary to have egress rules as strict as
ingress rules, but there are some things you may still want to filter in egress.

### Stop spoofing of outgoing traffic

To prevent someone on the network from spoofing outgoing traffic, you can set up
rules that only allow traffic from the internet-routable subnets you control.

#### /etc/pf.conf

```sh
table <routablenets> const { $ext_if:network, $int_if:network }

block drop out quick on $ext_if from ! <routablenets>  set prio 0
```

### Only MXes may send mail

To avoid errors and server problems causing us to send spam, and to ensure that
all outgoing mail goes via our mail server, we drop all outgoing SMTP traffic
from anything other than the mail servers.

#### /etc/pf.conf

```
block drop out quick on $ext_if proto tcp from ! <mail> to port 25 set prio 0
```

### Authpf

`authpf` is a form of network authentication that ties ssh and pf together. With
`authpf`, you can set up a shell account on the firewalls that friBytere can log
into to get access to services behind the firewall that are not available to the
outside world.

When a user whose shell is set to `authpf` logs in, predefined firewall rules
for the IP the user logs in from are loaded. The rules are defined in advance
and can be anything from access to port 80 on all machines to full access to
all ports on all machines behind the firewall. When the user logs out, all rules
loaded for that user and all firewall states are deleted immediately.

`authpf` makes it possible to run internal / test services behind the firewall
that are easily available to friBytere, but to nobody else.

Links

- [Hansteen authpf](http://home.nuug.no/~peter/pf/en/vegard.authpf.html)
- [OpenBSD authpf](http://www.openbsd.org/faq/pf/authpf.html)
- [authpf manual](http://www.openbsd.org/cgi-bin/man.cgi?query=authpf&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html)

### Rate limiting SSH

In addition to using log scrapers such as fail2ban, it can be useful to apply
other restrictions to SSH use against our servers. pf can place restrictions on
how many states an IP is allowed to have and how many states per second an IP is
allowed to create. This can be done for all firewall rules, but it is often best
suited for SSH.

Example

`pass in on $external to port 22 (max-src-conn 12, max-src-conn-rate 10/10, overload <sshbrutes> flush)`

- `max-src-conn 12` - no external machines (IPs) are allowed to have more than
  12 simultaneous connections
- ` max-src-conn-rate 10/10` - at most 10 new connections in 10 seconds
- `overload <sshbrutes> flush` - those that exceed the two rules above are added
  to the `<sshbrutes>` table and all their existing states on port 22 are
  removed (_state removal can also be done globally_)

Furthermore, you can choose what to do with the IPs in the `<sshbrutes>` table.
You can choose to block SSH for X minutes, block them completely for all
services, or for example put them in a traffic queue with very low priority and
little bandwidth. The example rule above will apply to all machines behind the
firewall, so at most 12 states can be created over SSH in total toward our
servers.

### spamd greylisting

To get rid of some spam and make our MX's job a bit easier, the routers should
run [spamd](http://en.wikipedia.org/wiki/Spamd). The plan is to run `spamd` in
greylisting mode with blacklists. The greylisting handles spammers that do not
run proper SMTP servers and/or do not follow the SMTP RFCs. "Nice" MXes are
whitelisted after about 1 hour and remain in the whitelist for 36 days (and
this is renewed each time the MX sends mail to us, so active MXes remain in the
whitelist).

In addition to greylisting, we can also explicitly whitelist MXes we do not want
to delay, such as `alf.uib.no`, `gmail.com`, `hotmail.com`, and so on. We can
also make use of explicit blacklisting of known spammers so that they always end
up talking to the tarpit instead of our MX. `spamd` ships preconfigured with
some good lists of spammers.

If combined grey- and blacklisting should not be enough, we can also set up
[greytrapping](http://en.wikipedia.org/wiki/Spamtrap) (aka spamtrap).

### Block IPv6 tunnels

To prevent machines on the NATed networks from setting up tunnels and causing
problems, we block the IPv6 tunneling technologies that can be blocked. This is
not a perfect solution; any usable network can be tunneled out of. This is only
to prevent automatic tunnels from causing problems.

```sh
block out quick on $ext_if inet proto 41 set prio 0
block out quick on $ext_if inet proto 47 set prio 0
block out quick on $ext_if inet to $sixtofour set prio 0
block out quick on $ext_if inet proto tcp to port $tsp
block out quick on $ext_if inet proto tcp to port $ayiya
block out quick on $ext_if inet proto udp to port $tsp
block out quick on $ext_if inet proto udp to port $ayiya
block out quick on $ext_if inet proto udp to port $teredo set prio 0
```

### Links:

- [Calomel spamd howto](https://calomel.org/spamd_config.html)
- [Hansteen spamd howto](http://home.nuug.no/~peter/pf/en/spamd.html)
- [spamd manual](http://www.openbsd.org/cgi-bin/man.cgi?query=spamd&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html)
- [spamd.conf manual](http://www.openbsd.org/cgi-bin/man.cgi?query=spamd.conf&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html)

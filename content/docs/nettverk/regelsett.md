+++
title = "Regelsett i produksjon"
description = "En gjennomgang av pf-regelsettet som er i produksjon"
template = "docs/page.html"
sort_by = "weight"
weight = 1
draft = false
+++

### Gjennomgang av regelsett

En gjennomgang av regelselsettet vi har i produksjon. Det kan godt være at dette
ikke er helt likt regelsettet som faktisk er i produksjon, men det burde vært
likt nok til å forklare hvordan regelsettet konstruert.

### Macroes

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

Regelsettet begyner med nyttige macroer (aka. variabler, referes til med $ som i
bash/perl...). Man bør heldst ha det meste av interfaces, IPer og porter som
macroer, det gjør regelsettet letter å vedlikeholde. Skulle man ha lyst til å se
hvordan regelsettet ser ut når alle macroer er evaluert og ekspandert kan man
kjøre

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

Under options er den noen viktige ting som skjer. Default policy for blokkering
av trafikk blir satt til return (standard er dropp), grunnen til at vi settere
dette er at det er lettere å feilsøke når man får beskjed at trafikk ikke får
slippe igjennom.

Loginterface angir hvilket interface pf skal samle statestikk for, det er vanlig
å settet det til **egress** interface-gruppen. Det interfacet som er koblet til
**default gateway** blir automagisk lagt til i **egress** interface gruppen.
Informasjonen som blir samlet kan sees ved å kjøre

```sh
# pfctl -si
```

**set skip on lo** betyr at pf ikke er aktivert for **lo** interface gruppen.
Denne gruppen inneholder alle loopback interfaces. Grunnen til at vi har denne
optionen med er at det svært sjeldent er nyttig å ha filtrering på loopback.

**set reassemble** sørger for at pakker som er fragmentert blir samlet opp og
satt sammen før de blir testet mot regelsettet. Dette sørger for at man ikke
skal kunne snike pakker gjennom brannmuren ved å bruke fragmenterings-triks.

**set optimization normal** setter alle timeouts for tcp og lignende til
"standard verdier".

**set timeout { tcp.established 3600, tcp.closing 60 }** setter ned noen av
timeout verdiene for tcp siden standard verdiene er mege høye.

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

Her defineres flere tabeller vi skal bruke for å gjøre ingress filtreringen
enklere. De første tabellene blir lastet inn fra filer under
**/etc/pf.d/tables**, disse filene er lister med IP addresser. Tabellene vi
definerer skal brukes senere i regelsettet for å slippe inn trafikk på
forskjellige tjenester (ssh, http ...) til de serverene som skal servere
tjenestene det gjelder.

De midterste tabellene (martians, sshguard og spamd) er tabeller som inneholder
IPer og nettverk vi vil blokkere. **Martians** inneholder alle IP addresser man
aldri skal motta trafikk fra på internett, som feks. private IP addresser,
multicast addresser og lignende. **ssguard** og **spamd** tabellene er begge
applikasjons tabeller, det betyr at innholdet i tabellene blir styrt av
"eksterne" programmer. sshguard er et program som scanner ssh loggene og legger
addresser som er for aggressive i **sshguard** tabellen slik at vi kan droppe
trafikk fra de gjeldene IPene. **spamd** tabellen blir oppdatert av programmet
spamd som laster ned lister over kjente spammer IPer og legger det i **spamd**
tabellen slik at ingen av de addressene får snakke med mailserveren vår.

De nedersete tabellene brukes for å forenkle regler hvor man vil matche trafikk
som skal rutes til internett og ikke internt. For å gjøre dette lister man alle
lokale nett i tabell også bruker man negasjonen av denne tabellen i regelen.
Feks.
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

Dette er reglene som NATer for Kvarteret sine to nett (klient og kasse). Disse
reglene gjør kun NAT, de sier ikke noe om trafikken blir sluppet igjennom eller
ikke, det må gjøres med egne **pass**-regler i tillegg til disse
**match**-reglene.

**match out on $ext_if** betyr at denne regelen matcher trafikk som skal ut på
det eksterne interfacet. **inet from $kasse_if:network** betyr at regelen
matcher IPv4 trafikk fra kassenettverket til Kvarteret. `to !<localnets>` gjør
at regelen kun matcher trafikk som skal til internett. **received-on $kasse_if**
sjekker at trafikken ble mottat på rett interface, dette er for å forhindre at
regelen ikke kan matche trafikk fra andre interfacer. **nat-to $kasse_ip** angir
hvilken offentlig IP trafikken skal mappes til.

OBS!! Alle IPer som blir brukt i en **nat-to $IP** klausul må være konfigurert
på brannmurene, de må være alias addresser på det carp interfacet som er koblet
til servernettet.

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

Brannmurreglene begyner med regler som spesifikt blokkerer og dropper uønsket
trafikk.

**urpf-failed** betyr trafikk som kommer inn på feil interface i forhold til
IPen det kommer fra, feks det kommer trafikk fra UiB fra Kvarteret sitt
nettverk.

**no-route** forklarer seg selv, den matcher trafikk vi ikke har noen route til.
Feks trafikk fra private IPer som ikke brannmurene er koblet til.

`<martians>` er tabell over alle IPer man aldri skal få trafikk fra, all trafikk
fra IPene listet i denne tabellen blir droppet på vei inn på det eksterne
interfacet.

De siste tre reglene sørger for at vi alltid får kontakt med brannmurene via ssh
og icmp, disse reglene er skrevet nokså generelle slik at de skal matche på alle
interfacer. ' Den siste regelen sørger for at vi alltid får sendt trafikk fra
brannmurene selv.

Linjen som bare inneholder **block** er det vi kaller standard blocken. Denne
regelen sørger for at all trafikk som ikke blir eksplisitt droppet eller sluppet
igjennom av andre regler blir blokkert med reset. Da tcp reset for tcp koblinger
og icmp reset for alle andre koblinger.

**quick**-kodeordet betyr at når noe trafikk matcher disse reglene skal man ikke
evaluere resten av regelsettet, men stoppe slik man gjør i iptables.

**self:0** matcher alle addresser som er konfigurert på brannmurene, utenom
alias addresser. Hadde vi bare brukt **self** hadde reglene også matchet alias
addresser, noe som ville gjort at brannmurene vil svare på ssh og icmp på IP
addressene som blir brukt for Kvarterts klient og kassenett.

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

### Kasse Interface

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

### Klient Interface

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

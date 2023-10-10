+++
title = "Redundant brannmur"
description = "Informasjon om brannmuren til friByte"
date = 2022-03-06T18:22:00+00:00
updated = 2022-03-06T18:22:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 1
draft = false
+++

**Kontaktperson:** Mikal Sande <mikal.sande@gmail.com>

Redundant brannmurprosjektet er ferdig, denne siden brukes til dokumentasjon av
oppsettet. Dole og Doffen er det som blir brukt som brannmurer / rutere.

### Redundans og failover

Siden brannmuroppsettet vårt består av to redundante brannmurer er det greit å
vite om noen kommandoer for administrering av et slikt oppsett.

### Hvem er master?

Det første man gjerne har lyst til å vite er om brannmuren man har logget inn på
er master eller slave.

```sh
# ifconfig carp
```

Viser alle interfaces som er i carp interface-gruppen. Den brannmuren som er
master skal ha ''master'' status på alle carp interfaces. Tilsvarende har slaven
''slave'' status.

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

### Endre master

Hvis man vil endre master kan man påtvinge en såkalt ''graceful failover'' ved å
kjøre

```sh
# ifconfig -g carp carpdemote 100
```

Man setter det tilbake ved å kjøre

```sh
# ifconfig -g carp -carpdemote 100
```

Hvis man skal skrue av masteren kan man enten kjøre kommandoen over eller bare
skrue av maskinen. Carp er innebygget i operativsystemet og vil sørge for å
gjøre en skikkelig failover når man skruer den av.

Hvis man restarter masteren vil den ta over som master når den kommer opp igjen,
den vil kun gjøre dette hvis den har fått synket all informasjonen den trenger
med slaven (som var master mens masteren var nede).

### Endre master over SSH

Hvis man skal endre master over ssh må man koble til de eksterne IPene på
'''begge brannmurene'''. Koblinger til de intern addressene eller master
addressen vil bli brutt når man endrer master.

```sh
ssh root@dole.uplink.ss.uib.no
ssh root@doffen.uplink.ss.uib.no
```

### Holde configer i sync

Alle config endringer skal gjøres på master (uplink.ss.uib.no) brannmuren, som
under vanlig drift skal være dole.ss.uib.no

Eksempel på config endring på begge brannmurene

```sh
cd /etc/
vi dhcpd.conf
scp dhcpd.conf doffen:$PWD
```

### Holde regelsettet i sync

pfsync protokollen sørger for at alle states blir syncet mellom brannmurene og
carp sørger for at brannmurene kan dele addresser og opptre som en brannmur. For
å holde regelsettet selv i sync har vi et selvlagd script som syncer ting for
oss når vi har gjort endringer.

### Fremgangsmåte

```sh
vi /etc/pf.conf
sh /root/syncpf.sh
```

### Legge til eller fjerne IPer fra ssh tabellen

```sh
vi /etc/pf.d/tables/ssh
sh /root/syncpf.sh
```

Sync-scriptet sjekker at regelsettet er gyldig og vil ikke synce et regelsett
som er ugyldig. Sync scriptet kopierer over **/etc/pf.conf** og **/etc/pf.d/**
til den andre brannmuren og så vil det laste inn det nye regelsettet på begge
brannmurene.

### Brannmur / Ruter Oppsett

Her følger en dokumentasjon og forklaring av hva som kjører på brannmurene, gitt
i rekkefølge prioritert etter hvor viktig det er for nettverket til friByte.

### Nettverksoppsett

Først setter man opp all nettverkskortene på brannmurene, først de fysiske
nettverkskortene og så de logiske. Alle nettverkskort-instillinger i OpenBSD
blir lagt i egne filer kalt hostname.if (hvor if er navnet på interfacet) under
/etc/.

### Fysiske nettverskort

/etc/hostname.em2 (uplink til UiB)

```sh
inet 158.37.2.19 255.255.255.248
dest 158.37.2.17
group uplink
group ingress
description "Uplink til UiB"
```

/etc/hostname.em1 (Server nettverk)

```sh
inet 158.37.6.32 255.255.255.192
group servers
description "server nettverk"
```

/etc/hostname.em0 (vlan trunk for klient, kasse og management)

```sh
up
description "vlan trunk"
```

/etc/hostname.bge0 (sync interface for pfsync og dhcp-sync)

```sh
inet 10.7.7.1 255.255.255.0
group pfsync
description "pfsync interface"
```

Litt info

- **em2** er vår uplink mot UiB og siden det er koblingen til brannmurene sin
  default gateway befinner seg på legger man til **dest 158.37.2.17**. Denne
  IPen legges også i filen **/etc/mygate**.
- **em1** er nettverkskortet som kobles til servernettet.
- **em0** har ikke noen IP fordi det interfacet brukes som VLAN trunk for litt
  diverse nettverk som vi kommer tilbake til.
- **bge0** er satt opp med privat IP addresse (RFC1918) fordi det er interfacet
  som blir brukt for å synce state-tabellen mellom brannmurene. Det er denne
  syncingen som gjør at brannmurene kan ta over for hverandre uten at noen
  nettverks-sesjoner blir avbrutt.

### Logiske nettverkskort

De logiske nettverkskortene settes opp på samme måte som de fysiske, med en
liten forskjell. Navnet som gis til et logiske nettverkskort bestemmer
funksjonen og man kan gi et logisk nettverkskort hvilket nummer man vil. Noe vi
har tatt i bruk for å gjøre det lettere å huske assosiasjonene mellom de logiske
og nettene de tilhører.

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

Vi har tre VLAN som kommer inn til brannmurene via **em0**. Etter et management
nett som ikke skal ha noe internett tilgang. De to andre er nett som vi skal
kjøre NAT (Network Address Translation) på for Kvarteret.

### CARP

Nå som alle de fysiske og logiske nettverkene er satt opp er det på tide å sette
opp de logiske interfacene for de IP addressene brannmurene skal dele. Det er
her CARP kommer inn. Når man bruker CARP trenger man minst tre IPer for ruterene
på alle nettene de skal være en del av. De trenger hver sin IP og en felles IP.
Når alt er oppe og kjører skal det kun være master brannmuren som bruker felles
IPen.

/etc/hostname.carp1 (server nett)

```sh
vhid 1 pass derpderpderp carpdev em1 advskew 0 158.37.6.33 netmask 255.255.255.192
alias 158.37.6.11 netmask 255.255.255.192
alias 158.37.6.30 netmask 255.255.255.192
description "server nett, felles IP"
```

/etc/hostname.carp2 (uplink til UiB)

```sh
vhid 2 pass roflrofllolroflrofl carpdev em2 advskew 0 158.37.2.18 netmask 255.255.255.248
description "uplink, felles IP"
```

/etc/hostname.carp3 (kasse nett)

```sh
vhid 3 pass roflmao carpdev vlan1 advskew 0 10.250.1.1 netmask 255.255.255.0
description "kasse, felles IP"
```

/etc/hostname.carp4 (klient nett)

```sh
vhid 4 pass w00t carpdev vlan10 advskew 0 10.250.10.1 netmask 255.255.254.0
description "klient, felles IP"
```

/etc/hostname.carp5 (management nett)

```sh
vhid 5 pass  carpdev vlan200 advskew 0 10.250.200.1 netmask 255.255.255.0
description "management, felles IP"
```

**carp1** skiller seg litt ut her, den har noen alias addresser fordi
brannmurene må selv eie addressene som blir brukt til NAT for kasse og klient
nettene, ellers kan det oppstå problemer.

Resten av oppsettet av CARP interfacene er likt. **vhid** er bare IDen til
interfacet.

- **pass** er passordet som blir brukt til å kryptere og autentisere alle CARP
  pakker mellom brannmurene for det gjeldende interfacet. Passordene er byttet
  ut med "morsomheter" fordi det er dårlig stil å ha passord liggende på Wikien
  :)
- **carpdev** er navnet på interfacet dette carp interfacet skal bruke.
- **advskew** sier noe om hvor ofte brannmuren gir beskjed at den er oppe. Den
  brannmuren som gir beskjed oftest blir master, derfor er det viktig at
  brannmurene settes opp med forskjellig **advskew**.
- Og resten av oppsettet med IPer og netmasker må matche instillingene på
  interfacet som er **carpdev**.

VIKTIG!! **avdskew** må settes forskjellig på brannmurene. Den man vil skal være
master bør ha **advskew** satt til **0** for alle sine CARP interfacer. På den
andre brannmuren kan man sette **advskew** til **100**.

OGSÅ VIKTIG!! Passordene som brukes på de forskjellige carp interfacene bør være
forskjellige og vanskelige og knekke. Siden dette passordet kun skal deles
mellom brannmurene kan man lage et sterkt passord ved å kjøre

```sh
openssl rand -base64 40
```

### sysctl

Når nettverkskortene er satt opp er det et par sysctl'er vi må endre på for at
brannmurene skal kunne "forwarde" trafikk. Først gjør vi endringen med
''sysctl'' kommandoen, så legger vi den til i ''/etc/sysctl.conf'' for å gjøre
endringen varig.

#### For å kunne forwarde IPv4 trafikk

```sh
# sysctl net.inet.ip.forwarding=1                                                                                    net.inet.ip.forwarding: 0 -> 1
```

/etc/sysctl.conf (fjern kommentaren fra linjen)

```sh
net.inet.ip.forwarding=1
```

#### For å kunne forwarde IPv6 trafikk

```sh
# sysctl net.inet6.ip6.forwarding=1
net.inet6.ip6.forwarding: 0 -> 1
```

/etc/sysctl.conf (fjern kommentaren fra linjen)

```sh
net.inet6.ip6.forwarding=1
```

#### For at carp skal fungere slik vi vil

```sh
# sysctl net.inet.carp.preempt=1                                                                                     net.inet.carp.preempt: 0 -> 1
```

/etc/sysctl.conf (fjern kommentaren fra linjen)

```sh
net.inet.carp.preempt=1
```

#### Fordi vi har fire fysiske nettverkskort

```sh
# sysctl net.inet.ip.ifq.maxlen=1024                                                                                 net.inet.ip.ifq.maxlen: 256 -> 1024
```

/etc/sysctl.conf (legg til linjen)

```sh
net.inet.ip.ifq.maxlen=1024
```

### pf

pf er brannmuren som følger med i OpenBSD, navnet er forkortelse for ''packet
filter''. For at brannmurene skal begyne å rute trafikk trenger man ikke noe
stort regelsett. Man kan begyne med et helt enkelt ett også legge til ting
etterhvert.

#### Minimalt regelsett

/etc/pf.conf

```sh
pass
```

Dette gjør at brannmurene oppfører seg som rutere. Servernettet vil fungere
fint, men ikke kasse og klient nettene.

#### Minimalt regelsett med NAT

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

Med dette regelsettet vil vi ha et fungerende nettverk uten noen brannmur
filtrering.

### Regelsett

For å gi et godt innsyn i hvordan pf oppsettet på brannmurene funger er det
blitt laget egne sider for hvordan man går frem for å skrive regelsett og en
gjennomgang av regelsettet som er i bruk.

[Gjennomgang av regelsettet som er i produksjon](/docs/nettverk/regelsett)

[Fremgang for å skrive regelsett](/docs/nettverk/regelsett-tutorial)

### Ideer som brukes for å forenkle regelsettet

Det er flere måter man kan skrive regelsett på i pf, måten vårt regelsett er
skrevet på er ment tå gjøre regelsettet forståelig og enkelt å endre på. For å
gjøre dette bruker vi flere ideer / teknikker som forenkler arbeidet.

### block by default

All innkommende trafikk blir blokkert og resatt med mindre det finnes en regel
som eksplisitt tillater trafikken. Av utgående trafikk (til Internett) tillater
vi det meste, med et par unntak.

Blokkerer all tafikk som ikke blir eksplisitt tillatt.

```sh
block log (to pflog1) set prio 0
```

### pf tables

For å gjøre filtrering av innkommende trafikk lettere bruker vi ''tables''.
Disse er gjerne lagret i filer under ''/etc/pf.d/tables'' og er kun lister med
IPer. Vi har lister til de forskjellige tjenestene vi har tilgjengelig, så det
er en liste for ''http'', ''ssh'' osv. Så hvis en serverer skal kjøre en
tjeneste som skal være tilgjengelig via Internett er det bare å legge IPen til
serveren inn i den rette listen også reloade brannmur reglene.

Slipper igjennom trafikk fra Internett vil våre webservere.

```sh
table <http> persist file "/etc/pf.d/tables/http"
pass in log on ingress inet proto tcp to <http> port $http_ports tag SERVER_IN set prio (4, 6)
pass out on $int_if tagged SERVER_IN modulate state (pflow) set prio 4
```

### Interface groups

Nettverket vårt har flere ingresser og kun en egress (uplink til UiB), dette
gjør at vi gjerne må tillate trafikk inn til serverene våre fra flere nettverk,
feks. fra Internett, Kvarterets klient nett og Kvarterets kasse nett. For å
unngå at vi må duplisere flere brannmurregler for å slippe inn samme trafikk fra
disse nettverkene bruker vi **interface groups**.

Med interface groups kan vi legge flere nettverkskort inn i samme gruppe, feks
**ingress** gruppen. Dette kan vi gjøre med alle interfaces som trenger å sende
trafikk til serverene våre (Internett, Kvarteret klient, osv...) og på den måten
trenger vi kun skrive reglene for hva vi vil slippe inn til serverene våre en
gang. Dette sparer oss for mye duplisering av regler og unødvendig regelsett
vedlikehold.

Slipper igjennom trafikk til minecraft serveren fra alle ingress nettverkene
våre. (Internett, klient, kasse og public)

```sh
happy = "158.37.6.57"
minecraft_port = "25565"
pass in log on ingress inet proto tcp to $happy port $minecraft_port tag SERVER_IN
pass out on $int_if tagged SERVER_IN modulate state (pflow) set prio 4
```

### tagging

Når trafikk går igjennom en ruter må tafikken gå igjennom minst to
nettverkskort. For en brannmur betyr det at all trafikk må evaluere opp mot
regelsettet to ganger før den slippes igjennom. Dette kan fort lede til mye
unødvendig duplisering av regler. Så istedet for å vedlike holde nokså like
regler flere plasser i regelsettet kan vi bruke **tagger**. Man tagger trafikk
når den kommer inn til brannmuren og bruker taggen til å slippe trafikken
igjennom på vei ut av brannmuren.

Slipper igjennom web, mail, dns og ssh fra alle ingress nettverkene (Internett,
klient, kasse og public) til serverene våre.

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

Spamd er en del av standard installasjonen, man trenger ikke installere noen
pakker.

#### /etc/rc.conf.local på dole

```sh
spamd_flags="-Y bge0 -y bge0"
spamlogd_flags="-Y 10.7.7.2"
```

#### /etc/rc.conf.local på doffen

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

#### Starte spamd for første gang

```sh
pfctl -f /etc/pf.conf
/etc/rc.d/spamlogd start
/etc/rc.d/spamd start
/usr/libexec/spamd-setup
```

### Hvitelisting med SPF og MX

For å hviteliste enkelte domener bruker vi et par scripts til å parse SPF
records for domenene. Scriptene henter ut IPer og legger dem i en table i pf
slik at vi kan slippe inn mail fra domenet uten å gå igjennom greylisting. Når
vi så mottar mail fra disse domene vil de bli hvitelistet av spamd, så dette er
for det meste en bootstrap teknikk.

MX records for domene blir også lagt til i hvitelisten.

Listen over domene som blir hvitelistet ligger i **/root/spamd_whitelist.sh**.

Scriptene

```sh
/root/spamd_whitelist.sh
/root/spf_dump.py
```

Crontab

```sh
10      1       *       *       *       /bin/sh /root/spamd_whitelist.sh
```

### Innkommende hvitelisting

**-I** gjør at spamlogd kun hvitelister addresser basert på innkommende smtp
trafikk. Det kan være litt problematisk å hviteliste utgående smtp trafikk på
grunn av bounces og lignende.

```sh
spamlogd_flags="-Y 10.7.7.2 -I"
```

### greyscanner

Greyscann er en daemon skrevet i perl som hvert 10ende minutt sjekker alle
greylistede entires i **spamdb** databasen. Hvis den finner noen entries som
ikke har A eller MX record i DNS **trappes** de i et døgn. Alle IPer som er
**TRAPPED** får kun snakke med **spamd** istedet for mailserverene våre.

Installere

```sh
# pkg_add -vi greyscanner
```

#### /etc/rc.conf.local

```sh
pkg_scripts="greyscanner"
```

### graytrapping

For å plage spammere enda mer er det satt opp
[greytrapping](http://home.nuug.no/~peter/pf/en/spamd.greytrapping.html) på
brannmurene. Ideen er at vi får spammere vil å skrape flere fiktive
mailaddresser fra nettsiden vår (fribyte.uib.no/mail/mail.txt), når de så prøver
å sende mail til disse addresse (som aldri skal motta legitim mail) kan vi trygt
svarteliste dem.

### Lokal blacklist

Vi har en lokal svarteliste vi kan legge til IPer i. Det er noen organisasjoner
som lever av å sende reklame for andre, slike organisasjoner kan vi like greit
blokkere fullstendig.

#### Legge til IPer i svartelisten

Legg til IPen du vil svarteliste i ''/etc/mail/blacklist.txt

#### Fjern IPer i svartelisten fra spamd databasen

```sh
sh /root/spamd-remove-blacklisted.sh
```

#### Restart spamd

```sh
/etc/rc.d/spamd restart
```

**Husk å gjøre endringene på begge brannmurene!**

```sh
cd /etc/mail
scp blacklist.txt doffen:$PWD
ssh doffen
  sh /root/spamd-remove-blacklisted.sh
  /etc/rc.d/spamd restart
```

## Notater

### dhcpd

dhcpd er en del av standard installasjonen, ingenting å installere.

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

ntpd er en del av standard installasjonen.

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

-s flagget sørger for å grovjustere klokken ved oppstart hvis klokken er
minutter eller mer feil.

### munin-node

#### installer

```sh
# pkg_add -vvi munin-node
```

så konfigurerer man som man føler for...

#### /etc/rc.conf.local

```sh
pkg_scripts="munin_node"
```

### sshguard

#### installer

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

Vi har ikke direkte backup på OpenBSD brannmurene. Istedet har vi et script
**/root/config-backup.sh** som kopierer over alle viktige configfiler over på
den andre brannmuren og bingen, som har backup mot UiB. Grunnen til dette er at
det er enklere å reinstallere OpenBSD og legge over de gamle config filene enn å
restore fra backup en komplett backup. Begge brannmurene kopiere filer inn i
hver sin katalog under **/home/config-backup** på bingen og under **/root** på
den andre brannmuren.

Reinstallasjon gjør man ved å reinstallere OpenBSD også kopiere over alle
config-filene fra bingen. Når den reinstallerte brannmuren kommer opp igjen skal
den være klar til å rute trafikk.

## Lenker

Howtos:

- [Presentasjon av PF](http://home.nuug.no/~peter/pf/eurobsdcon2012/index.html)
- [Innføring i PF](http://home.nuug.no/~peter/pf/en/)
- [Calomel Howto PF](https://calomel.org/pf_config.html)
- [Regelsett optimalisering](http://undeadly.org/cgi?action=article&sid=20060927091645)

### Viktige manualer

- [Manual: pf.conf](http://www.openbsd.org/cgi-bin/man.cgi?query=pf.conf&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html)
- [Manual: pfctl](http://www.openbsd.org/cgi-bin/man.cgi?query=pfctl&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html)

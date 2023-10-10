+++
title = "Fremgangsmåte for å skrive pf regelsett"
description = "En forklaring på hvordan man skriver et regelsett i pf"
date = 2022-03-06T18:52:00+00:00
updated = 2022-03-06T18:52:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 1
draft = false
+++

En liten fremgangsmåte for skriving av pf regelsett, andre howtos og kjennskap
til pf er greit å ha på forhånd. Den første delen er å regne som minimum av
arbeid som må gjøres for å få brannmurene til å fungere. Hvis det noen gang
skulle bli krise og vi må reinstallere eller sette opp maskinene på ny er det
den første delen som er viktigst, resten er "nice to have" og er ikke nødvendig
for at nettverket skal fungere.

De to viktigste funksjonene maskinen skal utføre er:

- Ruting
- Brannmur

Hvis det skulle være problemer og noe må fikses så skal disse funksjonene ha
prioritet.

### Få ting til å fungere

Før vil vi få ting til å fungere, la serverne våre kontakte den vide verden og
la Internet få kontakt med oss på de portene og tjenestene vi vil gi dem tilgang
til.

### Sett opp seksjoner for regelsettet

Dette er seksjonen vi deler regelsettet inn i, dette er noe pf forventer og det
gjør det hele strukturert.

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

### Først litt forarbeid

Vi starter med å fylle inn informasjon om nettverkskort, IPer, og lignende. Det
er greit å ha når vi skal skrive filter reglene. De options som er satt er
standard, vi setter hvilket interface vi vil logge statistikk for og hva vi skal
gjøre når noe trafikk skal blokkeres. Vi setter også opp vår første filter regel
som blokkerer alt som default. Når vi senere legger til flere regler vil de kun
slippe inn det de spesifiserer og alt annet vil bli blokkert.

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

### Slippe trafikk utenfra igjennom

For å få noe nytte utav brannmuren må den slippe trafikk igjennom. Det første vi
vil gjøre er å lage regler som slipper trafikk igjennom fra Internet til de
tjenestene vi kjører. Dette gjør vi per IP, så for at regelsettet ikke skal bli
altfor stort bruker vi tabeller for hver tjeneste. Tabeller gjør også
administrasjonen av regelsettet enklere. Hver tabell inneholder IPer vi har,
hver liste inneholder IPene til de serverne som kjører den tjenesten det
gjelder.

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

Disse reglene slipper igjennom trafikk fra Internet til de tjenestene vi kjører,
men det er en del gjentakelse i reglene for det eksterne og det interne
interfacet. Dette kan vi forenkle litt med bruk av tags. Tags i pf er kun en
intern makør pf bruker, og man kan filtrere med tags.

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

Ved hjelp av tags har vi forenklet regelsettet litt og gjort administrasjonen av
regelsettet noe enklere, bare husk å tagge ny regler når de legges til.

### Slippe trafikk innenfra igjennom

For å slippe trafikk innenfra ut gjør vi det samme som vi gjorde ovenfor,
forskjellen er at vi tagger på det interne interfacet og slipper ut ved hjelp av
tag på det eksterne interfacet.

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

### Rekkefølgen på reglene

For å gjøre regelsettet strukturert og effektivt er det greit å skrive reglene i
en viss orden. Første skritt er å ha egne seksjoner for hvert interface. Da er
det lettere å finne frem i regelsettet og det vil stemme bedre overens med
hvordan pf evaluerer regelsettet. Reglene over er ordnet etter interface,
retning og protokoll.

pf evaluerer regler i denne rekkefølgen: (skamløst hentet
[herfra](http://www.undeadly.org/cgi?action=article&sid=20060927091645))

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

Fordi pf evaluerer reglene i denne rekkefølgen er det greit å skrive regelsettet
i samme rekkefølge :) I de fleste tilfeller er det greit å følge de fire første
punktene, de fire siste er det ikke like mye å henta på kan ofte ignoreres.

### Hvor bør man filtrere hva?

Å skrive regelsett for en brannmur som også er ruter er noe annerledes enn å
skrive regelsett for en host som kan skal ha sin egen brannmur. En ting som kan
føre til forvirring er at en "pass" regel ikke garanterer at pakker kommer
igjennom, man må ha regler for både det innkommende interfacet og det utgående
interfacet for at trafikk skal komme igjennom. Vi har allerede fikset dette med
å bruke tags for å lette på arbeidet, men hvor skal forskjellige ting bli
filtrert?

En måte å gjøre det på er å gjøre all filtrering på ett interface, nedsiden med
dette er at noe trafikk gjerne får komme inn på ett interface men blir blokkert
når den skal ut. Det er egentlig ikke noen stor fare, men det kan være nyttig å
blokkere trafikk så tidlig som mulig.

En annen måte å gjøre det på er å alltid filtrere på det innkommende interfacet
og så bruke tags for å slippe trafikken ut på det utgående interfacet, dette er
slik reglene våre til nå ha blitt skrevet. _Helt ærlig er dette min (Mikal)
personlige preferanse, andre er gjerne uenige._

### NAT

Hvis vi skal gi internett tilgang til et klient nettverk (som Kvarteret) trenger
vi NAT. Heldigvis er NAT veldig enkelt i pf.

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

Filter reglene er allerede kjente, de eneste nye er match regelen. Match regler
kan stort sett ta i mot de samme argumentene som pass og block regler,
forskjellen er at match regler hverken blokkerer elle slipper igjennom. Det de
gjør er å utføre den spesifiserte handlingen på alle pakker som matcher regelen,
som i dette tilfellet er addresse oversetting i form av **nat-to $nat_ip**.

### Regler for brannmuren selv

Vi har fått trafikken i begge retninger i gang, samt NAT for et klient nettverk.
Nå er det på tide å differensiere mellom trafikk som skal gjennom brannmuren vs.
trafikk som skal til brannmuren. For det eksterne interfacet er dette nokså lett
siden vi allerede har brukt tabeller for å slippe inn trafikk. I første om gang
nøyer vi oss med å kun tillate ssh og ping inn til brannmuren. Her bruker vil
"self" nøkkelordet. Og siden vi vil at brannmuren alltid skal være tilgjengelig
på ssh og ping forenkler vi regelen slik at den gjelder for alle interfaces. Vil
også at all trafikk fra brannmuren skal slippe igjennom.

```sh
pass in quick inet proto tcp to self:0 port 22
pass in quick inet proto icmp to self:0
pass out quick inet from self:0
```

Når det gjelder server og klient nettverket må vi gjøre litt mer. Vi oppretter
en tabell, `<localnets>` over de lokale nettverkene ruteren er koblet til og
bruker denne tabellen til å differensiere mellom trafikk trafikk som har lokale
vs. globale endepunkt. Reglene som er kommentert ut er den gamle versjonen av
reglene.

```sh
# internal in
#pass in on $int_if inet from $int_if:network tag INT_OUT
pass in on $int_if inet from $int_if:network to ! <localnets> tag INT_OUT

# nat in
#pass in on $nat_if inet from $nat_if:network
pass in on $nat_if inet from $nat_if:network to ! <localnets>
```

### Regelsettet så langt

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

### Legge til litt mer funksjonalitet

Når konfigurasjonen i seksjonen over er fullført skal man ha et fungerende
regelsett som lar trafikk flyte i begge retninger. Denne seksjonen tar for seg
ting man fint kan legge til etter man har fått pf opp og kjøre.

### Blokkere litt fler ting

I tillegg til å ha en block by default policy er det greit å legge noen
eksplisitte blokkeringer av ting vi kan være sikre på at ikke er gyldig trafikk.
All trafikk som matcher disse block reglene skal droppes, det er ingen grunn til
å svare på ugyldig trafikk.

En kort liste

- [legges i en egen tabell](https://en.wikipedia.org/wiki/Martian_packet)
- Koblinger som bryter
  [urpf](https://en.wikipedia.org/wiki/Reverse_path_forwarding#Unicast_RPF_.28uRPF.29)
- IPer vi ikke har noen route til
- Koblinger som helt klart er spoofet (bruker nøkkelordet antispoof)

Til forskjell fra reglene vi har skrevet til nå skal disse reglene bruke "quick"
nøkkelordet. Quick gjør at pakker som matcher regelen ikke fortsetter evaluering
mot resten av regelsettet, quick gjør at regler oppfører seg som i ipfilter /
iptables.

```sh
antispoof quick for $ext_if

block drop in quick on $ext_if inet from urpf-failed set prio 0
block drop in quick on $ext_if inet from no-route set prio 0
block drop in quick on $ext_if inet from <martians> set prio 0
```

Antispoof er bare en snarvei, den ene regelen blir ekspandert til flere regler.
Feks. hvis `$ext_if` har IPen `158.37.6.4` vil `antispoof quick for $ext_if`
eksandere til

```sh
block quick drop in on ! $ext_if inet from 158.37.6.4/32 to any
block quick drop in inet from 158.37.6.4 to any
```

### egress filtre

Det kan være vanskelig og noe unødvendig å ha like strenge egress regler som
ingress regler, men det er enkelte ting man gjerne vil filtrere i egress.

### stopp spoofing av utgående trafikk

For å unngå at noen på nettverket spoofer utgående trafikk kan man sette opp
regler som kun tillater trafikk fra de internet-routable subnettene man
kontrollerer.

#### /etc/pf.conf

```sh
table <routablenets> const { $ext_if:network, $int_if:network }

block drop out quick on $ext_if from ! <routablenets>  set prio 0
```

### kun MXer får sende mail

For å unngå at feil og problemer med servere kan føre til at vi sender spam og
for å sikre at all utgående mail går via mailserveren vår dropper vi all
utgående smtp trafikk fra alle andre enn mailserverene.

#### /etc/pf.conf

```
block drop out quick on $ext_if proto tcp from ! <mail> to port 25 set prio 0
```

### Authpf

authpf er form for nettverks autentisering som kobler ssh og pf sammen. Med
authpf kan man sette opp en shell konto på brannmurene som friBytere kan logge
inn på for å få tilgang til tjenester bak brannmuren som ikke er tilgjengelige
for den vide verden.

Når en bruker, som har shell satt til authpf, logger inn vil det bli lastet inn
predefinerte brannmur regler for IPen brukeren logger seg inn fra. Reglene
defineres på forhånd og kan være alt fra tilgang til port 80 på alle maskiner
til full tilgang på alle porter på alle maskiner bak brannmuren. Når brukeren
logger av vil alle reglene som er lastet for den brukeren og alle brannmur
states bli slettet med en gang.

authpf gjør at man kan kjøre interne / test tjenester bak brannmuren som er lett
tilgjengelige for friBytere men ingen andre.

Lenker

- [Hansteen authpf](http://home.nuug.no/~peter/pf/en/vegard.authpf.html)
- [OpenBSD authpf](http://www.openbsd.org/faq/pf/authpf.html)
- [authpf manual](http://www.openbsd.org/cgi-bin/man.cgi?query=authpf&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html)

### Ratelimiting av ssh

I tillegg til bruk av loggskrapere som fail2ban kan det være greit å sette andre
restriksjoner på ssh bruk mot serverne våre. pf kan sette restriksjoner på
antall states en IP får lov til å ha og antall states per sekund en IP får lov
til å opprette. Dette kan gjøres for alle brannmur regler, men det passer gjerne
best for ssh.

Eksempel

`pass in on $external to port 22 (max-src-conn 12, max-src-conn-rate 10/10, overload <sshbrutes> flush)`

- `max-src-conn 12` - ingen eksterne maskiner (IPer) får lov til å ha flere enn
  12 samtidige koblinger
- ` max-src-conn-rate 10/10` - maks 10 nye koblinger på 10 sekunder
- `overload <sshbrutes> flush` - de som overskreder de to reglene ovenfor blir
  lagt til i `<sshbrutes>`-tabellen og alle deres eksisterende states på port 22
  blir fjernet (_state fjerning kan også gjøre globalt_)

Videre kan man velge hva man vil gjøre med IPene i <sshbrutes> tabellen. Man kan
velge å blokkere ssh i X antall minutter, blokkere dem helt for alle tjenester
eller feks. sette dem i en trafikk kø med veldig lav prioritet med lite
båndbredde. Eksempel regelen over vil gjelde for alle maskiner bak brannmuren,
så man får maks opprettet 12 states totalt over ssh mot serverne våre.

### spamd greylisting

Får å bli kvitt en del spam og gjøre jobben til MXen vår litt enklere skal
ruterne kjøre [spamd](http://en.wikipedia.org/wiki/Spamd). Planen er å kjøre
spamd i greylisting modus med blacklists. Greylistingen tar seg av spammere som
ikke kjører ordentlige SMTP servere og eller ikke forholder seg til RFCene for
SMTP, "snille" MXer blir whitelistet etter ca. 1 time og forblir i whitelisten i
36 dager (blir også fornyet hver gang MXen sender mail til oss, så aktive MXer
forblir i whitelisten).

I tillegg til greylisting kan vi også ha eksplisitt whitelisting av MXer vi ikke
vil forhindre feks, alf.uib.no, gmail.com, hotmail.com osv. Vi kan også ta i
bruk eksplisitt blacklisting av kjente spammere slik at de alltid får snakke med
tarpiten istedet for MXen vår, spamd kommer ferdig konfigurert med noen gode
lister over spammere.

Hvis kombinert grey- og blacklisting ikke skulle være nok kan vi også sette opp
[greytrapping](http://en.wikipedia.org/wiki/Spamtrap) (aka. spamtrap).

### blokkere IPv6 tunneller

For å unngå at maskiner på de NATede nettene setter opp tunneller og skaper
problemer blokkerer vi de IPv6 tunnellerings teknologiene som kan blokkeres.
Dette er ikke noen perfekt løsning, alle brukbare nettverk kan tunneleres ut av.
Dette er for å unngå at automatiske tunneller skaper problemer.

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

### Lenker:

- [Calomel spamd howto](https://calomel.org/spamd_config.html)
- [Hansteen spamd howto](http://home.nuug.no/~peter/pf/en/spamd.html)
- [spamd manual](http://www.openbsd.org/cgi-bin/man.cgi?query=spamd&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html)
- [spamd.conf manual](http://www.openbsd.org/cgi-bin/man.cgi?query=spamd.conf&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html)

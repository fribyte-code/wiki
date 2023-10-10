+++
title = "DNS"
description = "Informasjon om DNS-tjenesten til friByte"
date = 2022-03-06T17:11:00+00:00
updated = 2022-03-06T17:50:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 1
draft = false
+++

### DNS oppsett

Vi har to DNS servere, kladden er master og svartepetter er slave. Begge er satt
opp til å kjøre rekursive dns oppslag for servernettet (158.37.6.0/26). Alle
andre nett bruker kornelius som rekursiv DNS server.

### Oppsett

kladden er primary for alle domenene vi har DNS for og svartepetter er slave for
alle domenene vi har DNS for. I tillegg til dette er **alf.uib.no** slave for
alle uib.no domenene og **ns.hyp.net** slave for alle .no domenene.

Vi tillater dns zone transfers kun fra UiB (129.177.0.0/16), vårt subnet
(158.37.6.0/26) og Domeneshop sine dns servere (194.63.248.53, 194.63.255.6,
216.67.236.139). Grunne til dette er at zone transfers kan misbrukes til å
forsterke denial of servie angrep.

Når det gjelder rekursive oppslag godtar våre DNS servere rekursive oppslag fra
vårt interne nett (158.37.6.0/26). DNS serverene våre bruker ikke noen
**forwarders** etter UiB endret masse i sitt dns oppsett, de står i såkalt
"standalone" modus.

### Konfigurasjon

Alle dns soner vi har er lagt til i **/etc/bind/named.conf.local** og sonefilene
ligger under **/var/cache/bind/primary** på kladden og **/var/cache/bind/slave**
på svartepetter.

Etter vi fikk Kvarteret som kunde var det et behov for å ha noen dns-soner som
er interne, det vil si ikke offentlig tilgjengelige.

### Revers DNS

UiB vil ikke delegere videre revers dns for nettverk mindre enne /24, derfor må
vi sende mail til **hostmaster@uib.no** for å få gjort endringer i den
offentlige delen av revers DNS for vårt subnet.

Vi har satt opp revers dns på dns serverene våre slik at vi kan gjøre endringer
på "innsiden" uten å måtte vente på UiB.

Vi har ett /64 IPv6 til servernettet og vi har fått delegert revers dns for
dette subnettet fra Uninett. Hvis man må ta kontakt med Uninett angående noe
bruker man **drift@uninett.no**.

### friBytes soner

friByte har flere soner for egen bruk (ss.uib.no, fribyte.no, fribyte.uib.no).

- **ss.uib.no** brukes til maskiner og maskinavn på nettverket vårt, denne sonen
  skal ikke brukes av noen tjenester (ssh er et unntak). Derfor skal det ikke
  være noen CNAMEs fra andre soner til ss.uib.no domenet!
- **fribyte.uib.no** er brukt til tjenester og tjenestenavn. Her er det helt
  greit å bruke CNAMEs internt i sonen, det er også greit å bruke CNAMEs fra
  våre kunder soner mot denne sonen.
- **fribyte.no** bruker vi ikke noe særlig enda.

### Gjøre endringer

#### Hugs

- Når man legger inn nye domener i dns'en så må man huske å legge inn nytt
  seriellnr slik at andre dns'er merker at den har blitt oppdatert.
- Dersom ein berre skal legge til sub-domene, så eksisterar nok filene nedanfor
  allereie.

### Nye domener

For å legge inn nye domener i DNS må man:

- Logg inn på bind master serveren (ns.fribyte.uib.no)
- Lage eller endre zonefil /var/cache/bind/master/<domene.no>
- Legg til domenet og kontaktperson i /etc/bind/named.conf.local
- Eventuelt sett opp dnssec-nøkler under /var/cache/bind/keys/<domene.no>:
  `sh <pre>dnssec-keygen domene.no && dnssec-keygen -fk domene.no</pre> `
- Reloade bind serveren
- Logg inn på slave serveren (ns1.fribyte.uib.no)
- Oppdater /etc/bind/named.conf.local på slave serveren med den nye sona

**Zonefile** (/var/cache/bind/primary/<domene.no>) bør se ca slik ut for .no:

```sh
<pre>
$ORIGIN .
$TTL 7200        ; 1 hour
<domene.no>  86400   IN SOA  ns.fribyte.uib.no. hostmaster.fribyte.uib.no. (
                                2013102300 ; serial
                                3600       ; refresh (1 hour)
                                1800       ; retry (30 minutes)
                                3600000    ; expire (5 weeks 6 days 16 hours)
                                7200       ; minimum (2 hours)
                                )
        86400   IN      NS      ns.hyp.net.
        86400   IN      NS      ns.fribyte.uib.no.
        86400   IN      NS      ns1.fribyte.uib.no.
                        A       158.37.6.59
                        MX      10 mail.fribyte.uib.no.
</pre>
```

**Zonefile** (/var/cache/bind/primary/<domene>.uib.no) bør se ca slik ut for
.uib.no:

```sh
<pre>
$ORIGIN .
$TTL 7200        ; 1 hour
<domene>.uib.no       86400   IN SOA  ns.fribyte.uib.no. hostmaster.fribyte.uib.no. (
                                2013102301 ; serial
                                3600       ; refresh (1 hour)
                                1800       ; retry (30 minutes)
                                3600000    ; expire (5 weeks 6 days 16 hours)
                                7200       ; minimum (2 hours)
                                )
        86400   IN      NS      ns.fribyte.uib.no.
        86400   IN      NS      alf.uib.no.
        86400   IN      NS      ns1.fribyte.uib.no.
</pre>
```

Se [https://en.wikipedia.org/wiki/Zone_file wikipedia] for hjelp.

**named.conf.local** bør se ca sånn ut på kladden:

```sh
<pre>
zone "<domene.no>" {
        type master;
        file "primary/<domene.no>";
// Dersom DNSSEC:
//        key-directory "keys/<domene.no>";
//        auto-dnssec maintain;
//        inline-signing yes;
};
</pre>
```

**named.conf.local** bør se ca sånn ut på svartepetter:

```sh
<pre>
zone "<domene.no>" {
        type slave;
        file "slave/<domene.no>";
        masters { ss.uib.no; };
};
</pre>
```

### Endre domener

For å gjøre endringer på eksisterende domener logger man seg inn på master DNS
server som er kladden.

- gjøre endringer i zone filen (finnes i /var/cache/bind/primary)
- oppdatere **serial** til dagens dato
- så reloader man zonen som vist under

Husk å dobbeltsjekke med **dig** at endringen er gjennomført, test på begge DNS
serverene

```sh
<pre>
dig @158.37.6.52 <nyhost.domene.no>
dig @158.37.6.53 <nyhost.domene.no>
</pre>
```

**Reload** gjørest på denne måten:

```sh
rndc reload <domene.no> # Last inn endringene i sonefila
```

**Reload** gjørest på denne måten hvis det er en dynamisk sone:

```sh
 rndc reload             # Last inn nye sonefiler
 rndc freeze <domene.no> # Frys den dynamiske sona for endringar
 rndc reload <domene.no> # Last inn sona på nytt OBS! pass på at serielnummeret er endra
 rndc thaw <domene.no>   # Tin den dynamiske sona for endringar
```

### Bli kvitt .jnl filer

Bind oppretter binære .jnl filer for dynamsike soner som har mottatt
oppdateringer. For å dumpe .jnl filer inn i sonefilen igjen for en sone gjør man
på følgende måte.

```sh
<pre>
rndc freeze klient.kvarteret.no
rndc thaw klient.kvarteret.no
</pre>
```

### Aktive domener

friByte har mange domener. Disse ligger i **/var/cache/bind/primary** på kladden
og **/var/cache/bind/slave** på svartepetter.

### Globale soner

| Domenenavn                               | Kort forklaring                                 |
| ---------------------------------------- | ----------------------------------------------- |
| 1.0.0.0.1.0.2.0.0.0.7.0.1.0.0.2.ip6.arpa | Revers IPv6-sone for 2001:700:201:1::/64        |
| asf.uib.no                               | Aktive Studenters Forening                      |
| bstv.no                                  | [Bergen Student TV](./kunder/bstv)              |
| drageide.no                              | Vidar Drageide                                  |
| drageide.com                             | Vidar Drageide                                  |
| fagutvalget.no                           | Fagutvalget                                     |
| fuii.no                                  | Fagutvalget                                     |
| fribyte.no                               | friByte                                         |
| fribyte.uib.no                           | friByte                                         |
| heilhus.no                               | Kvarteret                                       |
| helhus.no                                | Kvarteret                                       |
| hulen.no                                 | Hulen                                           |
| iaeste.uib.no                            | IAESTE Bergen                                   |
| kvarteret.no                             | Kvarteret                                       |
| lanfear.org                              | Svein Harald Soleim                             |
| mikromandag.no                           | Kvarteret                                       |
| miljolisten.uib.no                       | Miljølisten                                     |
| oedipus.no                               | Ødipus                                          |
| skibas.no                                | Kvarteret                                       |
| sommerkvarteret.no                       | Kvarteret                                       |
| ss.uib.no                                | Gamle studentsenteret - brukes på våre maskiner |

### Lokale soner

| Domenenavn            | Kort forklaring                                             |
| --------------------- | ----------------------------------------------------------- |
| 6.37.158.in-addr.arpa | Revers-sone for 158.37.6/24                                 |
| 250.10.in-addr.arpa   | Revers-sone for 10.250.0.0/16                               |
| kasse.kvarteret       | Antakeligvis det gamle kassesystem-nettverket til Kvarteret |
| klient.kvarteret.no   | Uvisst                                                      |

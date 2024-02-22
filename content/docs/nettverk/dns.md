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

# Navnetjenere

Vi har to authorative navnetjenere:

- ns1.fribyte.no - 158.37.6.21 (master)
- ns2.fribyte.no - 158.37.6.22 (slave)

I tillegg har vi fantonald, som er vår rekursive navnetjener. Denne brukes for
våre servere og kunder som vi leverer nettverk til.

> I en overgangsperiode har vi også de gamle navnetjernerene. kladden og
> svartepetter og authoriative tjenere, og kornelius er rekursiv navnetjener.
> Disse vil bli brukt frem til vi får oppdatert navnetjenerene hos uib, srib og
> bstv.

## Oppsett av NS1 og NS2

NS1 og NS2 kjører bind9. En finner konfigurasjon i
`/etc/bind/named.conf.options` of `/etc/bind/named.conf.local`.

Sonefiler finnes under `/var/bind/master` på NS1. Her gjøres endringer for
soner.

Slave-serverene henter sonedata ved bruk av en transfer forespørser. NS1
tillater kun transfer til NS2 og ns.hyp.net. Dette bør også inkludere uib sin
navnetjener på et tidspunkt.

Hver gang NS1 sine sonefiler får en endring, vil den sende en notify til
slave-tjenerene, slik at de vil hente ut de oppdaterte sonene uten delay.

NS1 og NS2 er authorative navnetjenere. De vil ikke gjøre rekursive oppslag, og
vil kun gi svar om sine egne soner.

## Rekursiv DNS

Fantonald er vår rekursive navnetjener, som ersatter kornelius. Målet er at alle
maskiner under nettverk som Fribyte administrerer, skal bruke denne.

Fantonald kjører unbound, som er en enkel rekursiv DNS server som trenger lite
konfigurasjon. Den er satt opp til å kune svare til følgende nettverk:

- 158.37.6.0/25
- 10.250.0.0/16
- 2001:700:201:1::/64

## Revers DNS

UiB vil ikke delegere videre revers dns for nettverk mindre enne /24, derfor må
vi sende mail til **hostmaster@uib.no** for å få gjort endringer i den
offentlige delen av revers DNS for vårt subnet.

Vi har ett /64 IPv6 til servernettet og vi har fått delegert revers dns for
dette subnettet fra Uninett. Hvis man må ta kontakt med Uninett angående noe
bruker man **drift@uninett.no**.

> TODO: fikse revers IPv6 DNS for nytt oppsett

## friBytes soner

friByte har flere soner for egen bruk (ss.uib.no, fribyte.no, fribyte.uib.no).

- **ss.uib.no** fases ut. Ble før brukt for navn på egne maskiner og servere.
- **fribyte.uib.no** fases ut. Tidligere hoved-domene.
- **fribyte.no** brukes til det meste.

## Gjøre endringer

#### Hugs

- **Når man legger inn nye domener i dns'en så må man huske å legge inn nytt
  seriellnr slik at andre dns'er merker at den har blitt oppdatert.**
- Dersom ein berre skal legge til sub-domene, så eksisterar nok filene nedanfor
  allereie.

### Nye domener

For å legge inn nye domener i DNS må man:

- Logg inn på bind master serveren (ns1.fribyte.no)
- Lage eller endre zonefil /var/bind/master/<domene.no>
- Legg til domenet og kontaktperson i /etc/bind/named.conf.local
- Reloade bind serveren
- Logg inn på slave serveren (ns2.fribyte.no)
- Oppdater /etc/bind/named.conf.local på slave serveren med den nye sona

**Zonefile** (/var/bind/primary/<domene.no>) bør se ca slik ut for .no:

```zone
$ORIGIN fribyte.no.
$TTL 900
@       IN      SOA     ns1.fribyte.no. hostmaster.fribyte.no (
                                2024220201      ; serial (YYMMDDSS)
                                1h              ; refresh
                                15m             ; retry
                                2w              ; expiry
                                2h              ; minimum
                        )

        IN      NS      ns1.fribyte.no.
        IN      NS      ns2.fribyte.no.
        IN      NS      ns.hyp.net.
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

```
zone "fribyte.no" {
        type primary;
        file "master/fribyte.no";
        // Hvis DNSSEC:
        dnssec-policy default;
        inline-signing yes;
};
```

**named.conf.local** bør se ca sånn ut på svartepetter:

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

### Endre domener

For å gjøre endringer på eksisterende domener logger man seg inn på master som
er NS1.

- gjøre endringer i zone filen (finnes i /var/bind/master)
- oppdatere **serial** som ligger på linje 4
  - HUSK: følg formatet `YYMMDDSS`, der SS er 00, og økes med 1 dersom datoen er
    den samme
- Sjekk at sonefila er korrekt:

```
$ named-checkzone <domene> /var/bind/master/<domene>
```

- Reload

```
$ sudo rndc reload
```

Husk å dobbeltsjekke med **dig** at endringen er gjennomført, test på begge DNS
serverene

```sh
$ dig @158.37.6.21 <nyhost.domene.no>
$ dig @158.37.6.22 <nyhost.domene.no>
```

Alternativt bruk scriptet som gjør de fleste av stegene for deg.

```sh
$ ./update_zone.sh <domene>
```

### Bli kvitt .jnl filer

```sh
sudo rndc sync -clear
```

## Aktive domener

friByte har mange domener. Disse ligger i `/var/bind/master` på NS1.

### Globale soner

| Domenenavn                               | Kort forklaring                                           |
| ---------------------------------------- | --------------------------------------------------------- |
| 1.0.0.0.1.0.2.0.0.0.7.0.1.0.0.2.ip6.arpa | Revers IPv6-sone for 2001:700:201:1::/64 (mangler på nye) |
| bstv.no                                  | [Bergen Student TV](./kunder/bstv)                        |
| fribyte.no                               | friByte                                                   |
| fribyte.uib.no                           | friByte                                                   |
| ss.uib.no                                | Gamle studentsenteret - brukes på våre maskiner           |

### Lokale soner

| Domenenavn            | Kort forklaring               |
| --------------------- | ----------------------------- |
| 6.37.158.in-addr.arpa | Revers-sone for 158.37.6/24   |
| 250.10.in-addr.arpa   | Revers-sone for 10.250.0.0/16 |

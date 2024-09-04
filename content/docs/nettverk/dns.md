+++
title = "DNS"
description = "Informasjon om DNS-tjenesten til friByte"
template = "docs/page.html"
sort_by = "weight"
weight = 1
draft = false
+++

# Navnetjenere

Vi har to autorative navnetjenere:

- ns1.fribyte.no - 158.37.6.21 (master)
- ns2.fribyte.no - 158.37.6.22 (slave)

I tillegg har vi fantonald, som er vår rekursive navnetjener. Denne brukes for
våre servere og kunder som vi leverer nettverk til.

> I en overgangsperiode har vi også de gamle navnetjernerene. kladden og
> svartepetter og authoriative tjenere, og kornelius er rekursiv navnetjener.
> Disse vil bli brukt frem til vi får oppdatert navnetjenerene hos uib, srib og
> bstv.

> Mangler:
>
> - fribyte.uib.no
> - ss.uib.no
> - srib.no
> - bstv.no

## Oppsett av NS1 og NS2

NS1 og NS2 kjører bind9.

Sonefiler finnes under `/var/bind/master` på NS1. Her gjøres endringer for
soner.

Slave-serverene henter sonedata ved bruk av en transfer forespørser. NS1
tillater kun transfer til NS2 og ns.hyp.net. Dette bør også inkludere uib sin
navnetjener på et tidspunkt.

Hver gang NS1 sine sonefiler får en endring, vil den sende en notify til
slave-tjenerene, slik at de vil hente ut de oppdaterte sonene uten delay.

NS1 og NS2 er autorative navnetjenere. De vil ikke gjøre rekursive oppslag, og
vil kun gi svar om sine egne soner.

### Transfer

Ved en sone-endring i NS1, vil den sende en notify til alle slaver. Slavene vil
da gjøre en transfer, som vil si at de henter ut oppdaterte sonefiler fra
master. Hvilke IPer som har lov til å gjøre transfer er definert i
`named.conf.options`.

### Konfigurering

En finner konfigurasjon i `/etc/bind/named.conf.options` of
`/etc/bind/named.conf.local`, på både NS1 og NS2. NS2 er konfigurert til å være
slave.

Dersom en gjør en endring i disse filene, kjør alltid

    $ named-checkconfig

Ingen output tyder på at konfigurasjonen er gyldig. Deretter kan en kjøre

    $ sudo rndc reload

For å endre sonefiler, se lengre ned.

## Rekursiv DNS

Fantonald er vår rekursive navnetjener, som ersatter kornelius. Målet er at alle
maskiner under nettverk som Fribyte administrerer, skal bruke denne.

Fantonald kjører unbound, som er en enkel rekursiv DNS server som trenger lite
konfigurasjon. Den er satt opp til å kunne svare til følgende nettverk:

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

> TODO: fikse revers DNS for nytt oppsett

## Gjøre endringer for soner

### Hva er en sone?

Om du er litt i tvil på hva en sone egentlig er, les gjerne dette.
[https://ns1.com/resources/dns-zones-explained](https://ns1.com/resources/dns-zones-explained)

### Sonefil kort forklart

En sonefile består av flere resource records, som ser ca slik ut:

```zone
<navn>  <ttl>   IN      <type>  <data>
; Eksempel:
wiki    1h      IN      A       158.37.6.4
```

Dersom navn ikke er gitt, vil navnet til den over bli brukt. Dersom TTL ikke er
gitt, vil verdien fra $TTL variabelen bli brukt.

Noen vanlige typer:

| Type  | Bruk                                 |
| ----- | ------------------------------------ |
| A     | IPv4 adresse                         |
| AAAA  | IPv6 adresse                         |
| CNAME | Alias, peker til et annet domenenavn |

#### Hugs

- **Når man legger inn nye domener i dns'en så må man huske å legge inn nytt
  seriellnr slik at andre dns'er merker at den har blitt oppdatert.**
  - Serial nummeret må alltid være høyere enn det forrige.
- Dersom ein berre skal legge til sub-domene, så eksisterar nok filene nedanfor
  allereie.

### Endre domener

For å gjøre endringer på eksisterende domener logger man seg inn på master som
er NS1.

- gjøre endringer i zone filen (finnes i /var/bind/master)
- oppdatere **serial**
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

Et hjelpescript kan også brukes, som åpner en editor for sonen, sjekker
gyldighet og reloader sonen.

```sh
$ ./update_zone.sh <domene>
```

### Nye domener

For å legge inn nye domener i DNS må man:

- Logg inn på bind master serveren (ns1.fribyte.no)
- Lage eller endre zonefil /var/bind/master/<domene.no>
- Legg til domenet og kontaktperson i /etc/bind/named.conf.local
- Reloade bind serveren
- Logg inn på slave serveren (ns2.fribyte.no)
- Oppdater /etc/bind/named.conf.local på slave serveren med den nye sona

**Zonefile** (/var/bind/master/<domene.no>) bør se ca slik ut for .no:

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

**Zonefile** (/var/cache/bind/primary/<domene>.uib.no) bør se ca slik ut for
.uib.no:

Se [https://en.wikipedia.org/wiki/Zone_file wikipedia] for hjelp.

**named.conf.local** bør se ca sånn ut på NS1:

```
zone "fribyte.no" {
        type primary;
        file "master/fribyte.no";
        // Hvis DNSSEC:
        dnssec-policy default;
        inline-signing yes;
};
```

**named.conf.local** bør se ca sånn ut på NS2:

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

### Bli kvitt .jnl filer

```sh
sudo rndc sync -clear
```

## Aktive domener

friByte har mange domener. Disse ligger i `/var/bind/master` på NS1.

### Globale soner

| Domenenavn                               | Kort forklaring                                            |
| ---------------------------------------- | ---------------------------------------------------------- |
| 1.0.0.0.1.0.2.0.0.0.7.0.1.0.0.2.ip6.arpa | Revers IPv6-sone for 2001:700:201:1::/64 (mangler på nye)  |
| fribyte.no                               | friByte - hoved domene                                     |
| fribyte.uib.no                           | friByte - fases ut                                         |
| ss.uib.no                                | Gamle studentsenteret - brukes på våre maskiner - fases ut |
| bstv.no                                  | [Bergen Student TV](./kunder/bstv)                         |
| srib.no                                  | [Srib](./kunder/studentradioen)                            |

### Lokale soner

| Domenenavn            | Kort forklaring               |
| --------------------- | ----------------------------- |
| 6.37.158.in-addr.arpa | Revers-sone for 158.37.6/24   |
| 250.10.in-addr.arpa   | Revers-sone for 10.250.0.0/16 |

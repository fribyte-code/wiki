+++
title = "radio.srib.no"
description = "Oversikt over AzuraCast og IceCast2 som tilrettelegger for srib sin nettradio."
date = 2022-02-25T08:00:00+00:00
updated = 2024-01-11T08:00:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

## SRIB - AzuraCast & Icecast server

[AzuraCast](https://docs.azuracast.com/en/home) er et gratis åpen kildekode program for å drifte nettradio og annet. Den kommer innebygget med [Icecast server](https://icecast.org/) som brukes for å strømme lyd fra en eller flere kilder til mange klienter. For eksempel tusenvis av nettsidebesøkere. Dette brukes for å lage drifte [lytt.srib.no](lytt.srib.no) til SRIB i dag.

### Hosting

AzuraCast serveren hostes på VM srib-radio på konrad som en docker compose-instans. AzuraCast har en helt utrolig bra installer som er brukt til å sette opp systemet. [AzuraCast Docker installation guide](https://docs.azuracast.com/en/administration/docker/multi-site-installation). Filene ligger i mappen `/var/azuracast`. For å støtte å hoste flere docker containere på samme VM har vi brukt stegene under `AzuraCast Multi-Site Feature` som setter opp nginx-reverse-proxy og automatisk SSL sertifisering. Husk at andre docker containere må kobles til nettverket: `azuracast_frontend` for å at reverse proxy skal kunne se containerene.

#### Hoste statisk side

For å hoste en statisk side må du servere filene gjennom en http server som nginx. For eksempel [lytt.srib.no](lytt.srib.no) blir hostet ved at en statisk index.html ligger i mappen `/var/www/html/srib.no/lytt`. Og så har vi kjørt en nginx container som kun serverer filer fra denne mappen ved å mounte mappen med følgende kommando:

```sh
sudo docker run -d --restart always \
    --name lytt.srib.no \
    -v /var/www/srib.no/lytt:/usr/share/nginx/html \
    --network azuracast_frontend \
    --env VIRTUAL_HOST=lytt.srib.no \
    --env LETSENCRYPT_HOST=lytt.srib.no \
nginx:latest
```

- `--name` -> Navn på docker container
- `-v` -> Mount mappen med statisk filer til nginx sin default mappe
- `--network` -> Koble til AzuraCast sitt docker nettverk, påkrevd for å komme
  inn i reverse proxyen og for å få ssl sertifikater

AzuraCast har mange fancy funksjoner, disse kan utforskes på domenet til srib-radio vm'en. Av ting vi kanskje kan se på er å mounte srib-nasen til AzuraCast og sette opp automatisk avspilling av mediafiler når man ikke får inn noe signal på icecast serveren. Dette kan potensielt fjerne behovet for den dedikerte maskinen i SRIB sitt server-rom.

AzuraCast har og en podcast funksjon som kan være verdt å se på.

Det er og en [streamers & DJs](https://docs.azuracast.com/en/user-guide/streamers-and-djs) funksjon som ikke virker akkurat nå pga noe port trøbbel. Men som åpner for at flere kilder kan strømme inn lyd og så kan make trikse og mikse mellom strømmerne. Potensielt noe som kan brukes på utesendinger.

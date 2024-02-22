+++
title = "Studentradioen i Bergen"
description = "Informasjon om SRiB"
date = 2022-03-06T14:00:00+00:00
updated = 2022-03-06T14:00:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

Studentradioen i Bergen er en organisasjon på rundt 200 aktive medlemmer og
sender reklamefri radio 24/7. Grunnlagt i 1982, med røtter i flere passerende
forløpere siden 1950-tallet, er de nå den største radiokanalen driftet av og for
studenter, samt en av de siste FM-kanalene i Bergensområdet.

De er en hierarkisk organisasjon hvor det er et styre, utvalgt av
generalforsamlingen deres, som bestemmer. Under styret er redaksjonen og
driften. Redaksjonen, som også er valgt av generalforsamligen, er delt opp i 6
emne-redaksjoner og 1 kulisse-redaksjon, mens driften er delt opp i teknisk
gruppe, PR-gruppen og sosialkomitéen. Kontaktinformasjon er å finne nederst.

De er for øyeblikket en organisasjon som er ganske avhengig av tjenestene vi
leverer for dem for at de skal kunne være en radiostasjon. Per 06.03.2022 har de
skrevet under på vårt kontraktforslag, samt betalt. Den må fornyes i januar
neste år.

## Tjenester

| Tjeneste og domenenavn             | Kort Forklaring                                                                                                                                                                                               |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [srib.no](https://srib.no)                 | Wordpress-hovedside                                                                                                                                                                                           |
| [booking.srib.no](https://booking.srib.no) | Meeting Room Booking System - for å reservere rom                                                                                                                                                             |
| [skjema.srib.no](https://skjema.srib.no)   | LimeSurvey - for skreddersydde spørreundersøkelser                                                                                                                                                            |
| [wiki.srib.no](https://wiki.srib.no)       | BookStack - for å skrive og eie sin egen wiki                                                                                                                                                                 |
| [podcast.srib.no](https://podcast.srib.no) | RSS-generator skrevet i Django. Brukes for å lage RSS-strømmer til hver podcast. Kildekode: https://github.com/srib-dev/podkast.srib.no. Et docker image er bygget i srib-radio vm med secrets og kjører der. |
| [galleri.srib.no](https://galleri.srib.no) | PhotoPrism - bildegalleri for alle bilder radioen vil ta vare på                                                                                                                                              |
| [radio.srib.no](https://radio.srib.no)     | AzuraCast - Nettradiotjeneste bygget på IceCast og LiquidSoap                                                                                                                                                 |
| [lytt.srib.no](https://lytt.srib.no)       | Statisk nettside som serverer en ferdiglaget iframe-tag fra AzuraCast                                                                                                                                         |
| [Intern.srib.no](https://intern.srib.no)   | Gammelt lenketorg. Brukes fortsatt av produsentene deres                                                                                                                                                      |
| Database for digas                 | Digas er radiospesifikk programvare. Vertet i VM med navn "Database"                                                                                                                                          |

## Oppsett

### Nettsider

Alle tjenestene er vertet på "srib-radio" VMen som kjører på Proxmox clusteret.

- Tjenestene kjører i Docker containere
- Docker containerne er satt opp med docker-compose
- En docker compose fil per tjeneste

### Database for Digas

Vertet i en egen VM med navn "Database". Satt opp med en MySQL database rett i
Ubuntu.

## Typisk problemet

- "Srib.no virker ikke"
  - Sjekk om https://srib.no/wp-admin.php virker
  - "Database"-VM er full -> Øk diskplassen på VM-en
  - "srib-radio"-VM har gått i lås -> Restart VM-en
  - En av wordpress innleggene har en feil i seg -> Deaktiver nyeste innlegg og
    sjekk om det virker
- "Digas database er full"
  - "Database" VM-en er full som fører til at Digas og srib.no wordpress siden
    ikke virker. Øk disk størrelse
    [/docs/instrukser/full-disk](/docs/instrukser/full-disk/)

## Kontaktinformasjon

Sist oppdatert: 06.03.2022

| Stilling           | Navn                    | Nummer     | E-adresse                  |
| ------------------ | ----------------------- | ---------- | -------------------------- |
| Styreleder         | Magnus Røtnes           | I/T        | styreleder@srib.no         |
| Nestleder          | Erlend Haukeland Moe    | I/T        | nestleder@srib.no          |
| Ansvarlig Redaktør | Michael Fabregas Breien | 414 88 274 | ansvarlig.redaktor@srib.no |
| Daglig Leder       | Gruo Wågan              | 415 14 076 | daglig.leder@srib.no       |
| Teknisk Sjef       | Terje Peersen           | I/T        | terje.peersen@srib.no      |
| IT-ansvarlig       | Ingen                   | I/T        | I/T                        |
| PR-ansvarlig       | Bjørk Ellingsbø         | I/T        | pr@srib.no                 |

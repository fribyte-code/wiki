+++
title = "Studvest"
description = "Studvest"
date = 2023-01-12T15:51:00+00:00
updated = 2023-01-12T15:51:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

Studvest er studentorganisasjon som skriver aviser.

friByte har over lengre tid vertet en sambaserver uten å betale for tjenesten da verken fribyte eller Studvest visste hvor filene lå. Studvest skulle gå over til en annen filverter rundt 2015, men dette ble aldri skikkelig gjennomført, og filene ble dermed liggende på Skrue hele tiden. Tidligere hadde Studvest sin egen server; en Apple xServer frem til ca. 2013 hvor driften ble flyttet over til Skrue.

## Tjenester
- Nextcloud filverting

## Relevante domener

Sist Oppdatert: 12.01.2023

| Domenenavn                                       | Kort forklaring                   |
| ------------------------------------------------ | --------------------------------- |
| [studvest.no](studvest.no)                       | Hovedside, ikke vertet av friByte |
| [studvest-nc.fribyte.no](studvest-nc.fribyte.no) | Nextcloud server for Studvest     |

## Kontaktinformasjon

Sist Oppdatert: 12.01.2023

| Stilling           | Navn                  | Nummer     | E-adresse                     |
| ------------------ | --------------------- | ---------- | ----------------------------- |
| Ansvarlig Redaktør | Even Hæhre Hammersvik | 474 45 155 | ansvarligredaktor@studvest.no |


## NextCloud server

fribyte hoster NextCloud server hvor Studvest lagrer sine filer ca. 3TB (2022).

Oppsett:
- VM i Proxmox nettverket med navn `studvest` kjører docker container med nextcloud
- VM i Proxmox nettverket med navn `minnio` kjører Minio, en S3 kompatibel objekt lagrings tjeneste.
- Nextcloud er satt opp med en admin bruker som friByte kjører, passord ligger i https://bitwarden.friByte.no
- Studvest har en egen admin bruker `studvest-admin` som styrer gruppen `studvest`
- Nextcloud har koblet til minio serveren med `remote storage` plugin.

### Studvest prosedyre for å se filer på pc
1. Last ned NextCloud client [https://nextcloud.com/clients/](https://nextcloud.com/clients/)
2. Installer og åpne klienten
3. Definer "https://studvest-nc.fribyte.no" som server når de ber om det.
4. En nettside vil å åpne seg i nettleser og be deg logge inn
5. På neste side velger du hvor mappen skal ligge på pcen

### Studvest prosedyre for å invitere nye brukere:
1. Logg inn som studvest admin med brukernavn `studvest` og passord som dere har laget selv.
2. Klikk på ikonet oppe til høyre og klikk `Users`
3. Klikk `new user` oppe til venstre
4. Definer brukernavn, passord, email osv.
5. Klikk på `Add user to group` og velg `studvest` -> Brukeren vil nå få tilgang til den studvest mappen
6. Klikk `Add new user` og nå skal brukeren få en invitasjonsmail

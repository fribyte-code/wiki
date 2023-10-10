+++
title = "Passord - Bitwarden"
description = "Hvordan vi holder styr på passord"
date = 2022-03-19T17:00:00+00:00
updated = 2022-03-19T08:00:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 2
draft = false
+++

## Bitwarden passord verting

[https://bitwarden.fribyte.no/](https://bitwarden.fribyte.no/)

Vi holder styr på alle felles passord til diverse tjenester i vår selvvertede
Bitwarden instans.

Den ligger som en docker container på pengebingen og er startet opp med følgende
docker-compose som ligger i hjem-mappen til pengebingen.

```sh
version: '3'
services:
 vaultwarden:
  container_name: vaultwarden
  image: vaultwarden/server:latest
  restart: always
  volumes:
   - /vw-data/:/data/
  environment:
   - VIRTUAL_HOST=bitwarden.fribyte.no
   - LETSENCRYPT_HOST=bitwarden.fribyte.no
   - SIGNUPS_DOMAINS_WHITELIST=fribyte.no,fribyte.uib.no
   - ADMIN_TOKEN=SUPER-DUPER-SECRET-TOKEN
  network_mode: bridge
```

- Denne skal boote hver gang pengebingen booter, men om du vil endre
  innstillinger og iverksette de kan du skrive
  `sudo docker-compose -f vaultwarden-compose.yaml up -d`. Da vil containeren
  kreeres på nytt og alle innstillinger blir satt i bruk.

### Backup

Vi har per nå ingen backup av pengebingen, det må vi få ordnet, spesielt
bitwarden passordene. I pengebingen er det viktigste å ta backup av:

- Docker volumes
- Postgres databasen

Har vi backup av docker volumene så kan vi rimelig lett starte opp igjen alt på
en annen maskin ved å starte opp igjen docker containerne med samme
innstillinger.

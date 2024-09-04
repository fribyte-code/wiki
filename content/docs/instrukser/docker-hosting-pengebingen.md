+++
title = "Docker-hosting Pengebingen"
description = "Hvordan hoste docker containere"
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

## Pengebingen

[pengebingen.fribyte.no](https://pengebingen.fribyte.no) er en vm laget for å
hoste interne docker containere. Fordelen med docker containere er at de bruker
mye mindre ressurser enn VM'er og inkluderer alt som trengs uten at man må
innstallere alt mulig merkelige pakker for å få tjenesten til å virke.

Den er satt opp med letsencrypt renewal ved acme-companion
[nginx-proxy-acme companion](https://github.com/nginx-proxy/acme-companion).

Den har en automatisk nginx reverse proxy!

- Så det eneste man trenger for å ordne ssl sertifikat er å legge til
  `VIRTUAL_HOST=domain` og `LETSENCRYPT_HOST=doamin` som docker container env
  variabler.

## Sette opp ny nettside:

1. Lag et docker image av nettsiden du vil hoste
2. Sett opp subdomene ved å følge dokumentasjonen på
   [domener](@/docs/instrukser/domener.md). Sett domen den til å peke mot
   pengebingen.fribyte.no sin ip-addresse.
3. Kjør docker imaget på pengebingen som beskrevet på acme-companion siden:

```sh
sudo docker run --detach \
    --name {definer-pent-navn-her} \
    --restart always \
    --env "VIRTUAL_PORT=3000" \
    --env "VIRTUAL_HOST=subdomain.fribyte.no" \
    --env "LETSENCRYPT_HOST=subdomain.fribyte.no" \
    {docker-image}
```

- Det er viktig at `--restart always` er med for å sikre at containeren startes
  ved docker restart eller VM reboot.
- Virtual_host kan droppes om tjenesten i docker imaget allerede kjører på port
  80

4. Om DNS raden er blitt aktiv vil tjenesten være tilgjengelig på
   subdomain.fribyte.no ferdig fiksa med SSL sertifikat.

### Kommunikasjon mellom docker containere:

Om du trenger å snakke med en database for eksempel postgres må du koble
containeren til samme docker nettverk som postgres kjører på. Dette er i dag
`pengebingen-docker-network`. Da blir oppstart av docker containeren for
eksempel:

```sh
sudo docker run --detach \
    --name fribyte-ctf \
    --restart always \
    --env "VIRTUAL_PORT=8080" \
    --env "VIRTUAL_HOST=ctf.fribyte.no" \
    --env "LETSENCRYPT_HOST=ctf.fribyte.no" \
    --env "ADMIN_PASSWORD=PASSORD" \
    --env "DB_POSTGRES_CONNECTION_STRING=postgres://postgres:postgres@postgres:5432/ctf" \
    --network pengebingen-docker-network \
    mathiash98/fribyte-ctf
```

- Husk at når når man kommuniserer mellom docker containere bruker man ikke
  `localhost`, men heller hostname til hver container. Slik at tilkobling til
  postgres blir `postgres:5432` istedenfor `localhost:5432`.

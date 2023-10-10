+++
title = "Mattermost"
description = "Intern kommunikasjon"
date = 2022-03-12
updated = 2022-03-12
template = "docs/page.html"
sort_by = "weight"
weight = 3
+++

For all intern kommunikasjon bruker vi Mattermost.

## Obligatoriske kanaler

Nye medlemmer blir automatisk med i kanalene under, men pass på at du er med i
disse.

### `viktig`

Viktige beskjeder om møter, hendelser o.l. kommer her, få det med deg. Vi prøver
å være flinke på å bruke `@channel`.

### Ellers obligatoriske kanaler

```json
"ExperimentalDefaultChannels": [
  "generelt",
  "admis",
  "viktig",
  "ymse",
  "sosialt",
  "[drift] alt",
  "dugnad"
]
```

### Oppdater listen

1. Kjør `mmctl config edit`
1. Finn `ExperimentalDefaultChannels`
1. Legg til / fjern kanalen i arrayen.

## Kanalnavn (navnekonvensjon)

- Drifts relaterte kanaler skal ha `[drift]`-prefix.
- Kanaler som kun tar for seg Gitea-oppdateringer skal ha `[gitea]`-prefix.
- Kanaler som tar for seg statusendringer relatert til en eller flere servere
  skal ha `[drift:status]`-prefix.

Er du usikker, spør i `generelt`, vi kan alltids rename kanaler.

## Webhooks

Mattermost støtter å sende og motta webhooks, det er dette vi bruker for å få
statusoppdateringer fra Gitea.

Man kan aksessere disse ved å klikke oppe i venstre hjørne og så velge
**Integrations**.

## Gjøre konfigurasjoner direkte fra din egen maskin

Mattermost tilbyr et fint kommandolinjeverktøy,
[`mmctl`](https://docs.mattermost.com/manage/mmctl-command-line-tool.html).

1. Installer `mmctl`
1. Generer din egen token for din Mattermost-bruker
   1. Klikk på degselv oppe i høyre hjørne
   1. Klikk på **Profile**
   1. Velg **Security** fra sidepanelet
   1. Lag en **token**, ta vare på **Access Token**.
1. Kjør
   `mmctl auth login https://chat.fribyte.no --name fribyte --access-token <din-egen-token>`
1. Profit! Du kan nå bruke `mmctl` lokalt for å gjøre forskjellige greier.

## Fornye LetsEncrypt-sertifikat

Mattermost do currently <b>not have automatic certificate renewal</b>, following steps need to be performed until we fix an automatic cron job or install nginx-acme-companion. 

Steg:

1. Lag en sikkerhetskopi av `mattermost`-VM'en
2. SSH inn i VM-en etter sikkerhetskopien er ferdig
3. Kjør følgende kommandoer

```sh
sudo docker run --rm --name certbot \
    --network mattermost \
    -v "/home/fribyte/docker/certs/etc/letsencrypt:/etc/letsencrypt" \
    -v "/home/fribyte/docker/certs/lib/letsencrypt:/var/lib/letsencrypt" \
    -v shared-webroot:/usr/share/nginx/html \
    certbot/certbot renew --webroot-path /usr/share/nginx/html

sudo docker restart nginx_mattermost
```

Commands are fetched from:
- [https://docs.mattermost.com/install/install-docker.html](https://docs.mattermost.com/install/install-docker.html)
- [https://github.com/mattermost/docker/blob/main/docs/issuing-letsencrypt-certificate.md](https://github.com/mattermost/docker/blob/main/docs/issuing-letsencrypt-certificate.md)


4. Test om mattermost fortsatt fungerer ved å sende en melding


## Ting å være obs på

- Mattermost er installert på `konrad` (17.06.2023)
- VM-en kjører på `gjertrud` (17.06.2023)
- Mattermost er installert med
  **[Mattermost Omnibus CLI](https://docs.mattermost.com/install/installing-mattermost-omnibus.html)**
  - Da er ikke alle config-feltene i `/opt/mattermost/config.json` redigerbare
    (eller de vil ikke oppdatere seg). Se denne
    [lenken](https://docs.mattermost.com/install/installing-mattermost-omnibus.html)
    for mer informasjon.
  - Sjekk `/etc/mattermost/mmomni.yml` om det er noen felter som kan endres der
    før det gjøres i `config.json`.

## Oppdater Mattermost

1. Hent siste versjon av Mattermost  
	- `sudo docker pull mattermost/mattermost-enterprise-edition:latest`
2. Stopp Mattermost-containeren  
	- `sudo docker stop mattermost`
3. Sjekk om containeren "mattermost" er stoppa  
	- `sudo docker ps -a`
4. Sjekk om containeren "mattermost" er borte  
	- `sudo docker rm mattermost`
5. Hent siste versjon av Mattermost igjen  
	- `sudo docker pull mattermost/mattermost-enterprise-edition:latest`
6. Kjør scriptet `./start_mm.sh`

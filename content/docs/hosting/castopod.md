+++
title = "Castopod"
description = "Åpen kildekode podcast-tjener"
date = 2023-03-21T08:00:00+00:00
updated = 2023-03-24T08:00:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 2
draft = false
+++

friByte drifter sin egen podcast-tjener for å distribuere sin egen podcast, "Teknisk Tenketank". Til dette formålet bruker vi [Castopod](https://docs.castopod.org/) og er å finne på [podcast.fribyte.no](https://podcast.fribyte.no).

### Docker Compose filen

Jeg brukte en docker-compose fil, basert på [den som Castopod gir oss som standard](https://docs.castopod.org/getting-started/docker.html#example-usage). Jeg endret den selvfølgelig slik at den er tilpasset våre miljøer. 

Her er Docker Compose filen som fungerte til slutt:

```Docker
version: "3.7"

services:
  app:
    image: castopod/app:latest
    container_name: "castopod-app"
    volumes:
      - castopod-media:/opt/castopod/public/media
      - ttt_mount:/opt/castopod/public/media/podcasts/teknisktenketank

    environment:
      MYSQL_HOST: cpmariadb
      MYSQL_DATABASE: castopod
      MYSQL_USER: castopod
      MYSQL_PASSWORD: lelelelelel
      CP_BASEURL: "http://podcast.fribyte.no"
      CP_ANALYTICS_SALT: enellerannenstreng #laget med: tr -dc \!\#-\&\(-\[\]-\_a-\~ </dev/urandom | head -c 64 
      CP_CACHE_HANDLER: redis
      CP_REDIS_HOST: redis
      CP_DATABASE_HOSTNAME: castopod-mariadb
      VIRTUAL_HOST: podcast.fribyte.no
      LETSENCRYPT_HOST: podcast.fribyte.no
    networks:
      - proxy_network
      - castopod-db
    restart: always

  web-server:
    image: castopod/web-server:latest
    container_name: "castopod-web-server"
    volumes:
      - castopod-media:/var/www/html/media
      - ttt_mount:/var/www/html/media/podcasts/teknisktenketank

    networks:
      - proxy_network
    ports:
      - 8080:80
    restart: always
    environment:
      VIRTUAL_HOST: podcast.fribyte.no
      LETSENCRYPT_HOST: podcast.fribyte.no

  cpmariadb:
    image: mariadb:10.5
    container_name: "castopod-mariadb"
    networks:
      - castopod-db
    volumes:
      - castopod-db:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: lolololol
      MYSQL_DATABASE: castopod
      MYSQL_USER: castopod
      MYSQL_PASSWORD: lelelelelel
    restart: always

  redis:
    image: redis:7.0-alpine
    container_name: "castopod-redis"
    volumes:
      - castopod-cache:/data
    networks:
      - proxy_network

  # this container is optional
  # add this if you want to use the videoclips feature
#  video-clipper:
#    image: castopod/video-clipper:latest
#    container_name: "castopod-video-clipper"
#    volumes:
#      - castopod-media:/opt/castopod/public/media
#    environment:
#      MYSQL_DATABASE: castopod
#      MYSQL_USER: castopod
#      MYSQL_PASSWORD: lelelelelel
#    networks:
#      - proxy-network
#    restart: always

volumes:
  castopod-media:
  castopod-db:
  castopod-cache:
  ttt_mount:
    driver: vieux/sshfs:latest
    driver_opts:
      sshcmd: root@158.37.6.43:/dalar/podcast_ttt
      password: kekekekek
      allow_other: ""

networks:
  proxy_network:
    external: true
  castopod-db:
```

`ttt_mount` er et Docker volum som importerer filer som ligger lagret på Skrue (forklares nærmere litt under). Som du ser i definisjonen av volumet, brukes det SSHFS for å mounte filene. Passordet i denne konfigurasjonen blir ssh-passordet til `root`-brukeren på Konrad. 

En kan også benytte seg av ssh-nøkkel for verifisering. Det kan gjøres på denne måten:

```Docker
ttt_mount:
    driver: vieux/sshfs:latest
    driver_opts:
      sshcmd: root@158.37.6.43:/dalar/podcast_ttt
      IdentityFile: "/root/.ssh/ssh-key-file"
      allow_other: ""
```

Men i så tilfelle er det faktisk nødvendig at ssh-nøkkelen ligger i `/root/.ssh/..`. Hvis ikke, finner ikke `vieux/sshfs`-utvidelsen ssh-nøkkelen. [Kilde](https://github.com/vieux/docker-volume-sshfs/issues/58#issuecomment-447317661) 

Utvidelsen installerte jeg ved å kjøre: `docker plugin install --grant-all-permissions vieux/sshfs` i Pengebingen. 

Disse filene blir mountet direkte inn i Docker-containerne `castopod-app` og `castopod-web-server` fra Skrue. Det er viktig at filene er tilgjengelige i den respektive mappen i **begge** containerne.

Podcast-episodene som opprettes gjennom admin-panelet til podcast.fribyte.no, opprettes under `../media/podcasts/<navn-på-podcast>`. Derfor, når en episode nå lastes opp til gjennom admin-panelet, lagres den på Docker-volumet som er mountet fra Pengebingen inn i containerne. 

### Importere lydfiler

For å importere podcast-episodene som vi allerede hadde lagt ut gjennom Studentradioen i Bergen, brukte jeg Castopod sin innebygde importeringstjeneste. Den lastet inn alle episodene, samt metadata gjennom RSS-strømmen `https://podcast.srib.no/feed/254`. 

### Feilsøking

I første omgang, fikk jeg en `404`-feil i nettleseren når nettsiden prøvde å laste inn lydfilene og andre medfølgende bildefiler. Dette var fordi jeg bare mountet volumet inn i `castopod-app`-containeren, og ikke `castopod-web-server`-containeren.

Dette fikset jeg ved å mounte `ttt_mount` inn i `castopod-web-server` også. 

### Kilder:

[Castopod: Getting Started - Official Docker images](https://docs.castopod.org/getting-started/docker.html#example-usage), sist lest 24.03.2023.

[Mounting a SFTP (SSH) Share as a Volume in Docker-Compose](https://simplytim.io/mounting-a-sftp-ssh-share-as-a-volume-in-docker-compose/), sist lest 24.03.2023.

[How do debug when password-less mount does not work](https://github.com/vieux/docker-volume-sshfs/issues/58#issuecomment-447317661), sist lest 24.03.2023.
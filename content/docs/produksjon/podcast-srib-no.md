+++
title = "podcast.srib.no"
description = "Forklaring på hvordan podcast-tjenesten til studentradioen er rigget opp"
date = 2024-01-23T08:00:00+00:00
updated = 2024-01-23T08:00:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

### Koden

Koden er en hjemmesnekret Django-applikasjon, skrevet av en tidligere teknisk sjef hos studentradioen. [Du kan lese mer om koden her.](https://github.com/srib-dev/podkast.srib.no/blob/master/README.md)

### Docker compose fil

```yaml
version: '3'
services:
 srib-podcast:
  container_name: srib-podcast
  image: srib-podcast
  restart: always
  volumes:
   - type: bind
     source: /home/fribyte/srib-nas-mount/digasLydfiler/podcast
     target: /media/podcast
     read_only: true
  networks:
   - frontend
  environment:
   - VIRTUAL_HOST=podcast.srib.no
   - LETSENCRYPT_HOST=podcast.srib.no

networks:
 frontend:
  external: true
```

Det er en relativt enkel compose-fil, men det er verdt å merke seg:

```yaml
  volumes:
   - type: bind
     source: /home/fribyte/srib-nas-mount/digasLydfiler/podcast
     target: /media/podcast
     read_only: true
```

Denne blokken binder en mappe på studentradioen sitt lokale NAS, `/home/fribyte/srib-nas-mount/digasLydfiler/podcast`, inn i Docker-beholderen, i mappen `/media/podcast`.

### Fildeling

Om det ikke var åpenbart, så er 
Denne blokken binder en mappe på studentradioen sitt lokale NAS, `/home/fribyte/srib-nas-mount/digasLydfiler/podcast` en lokal mappe på VM-en. Innholdet i den mappen kommer fra studentradioen sitt lokale NAS, som står i deres serverrom, men er på samme subnett. 

Vi bruker CIFSv3 til å feste den relevante mappen fra NAS-et deres inn i VM-en vår, med denne linjen: `//158.37.6.118/NAS/     /home/fribyte/srib-nas-mount    cifs    vers=3.0,username=fribyte,password=kekekekek,noexec 0 0` i `fstab` 

Denne fildelingen krever at studentradioen godkjenner at vi kan koble oss til NAS-et deres. 
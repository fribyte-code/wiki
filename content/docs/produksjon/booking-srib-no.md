+++
title = "booking.srib.no"
description = "Forklaring på hvordan romreservasjonssystemet til radioen er skrudd sammen"
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

### Kort forklaring

Romreservasjonstjenesten vi leverer for radioen er en av de viktigste. Uten den,
blir det ikke mulig for dem å vite hvem av de 200 aktive medlemmene deres som
låner lokalene til hvilke tider.

Programvaren er [Meeting Room Booking System](https://mrbs.sourceforge.io/), og
kjører vanligvis i en LAMP stack. I vårt tilfelle kjører vi den gjennom Docker.

### Docker compose fil

```yaml
version: "2"
services:
  mrbs:
    image: dorianim/mrbs:latest
    container_name: mrbs
    environment:
      - PUID=1000
      - PGID=1000
      - DB_HOST=mrbs-db
      - DB_USER=mrbs-user
      - DB_PASS=lolololol
      - DB_DATABASE=mrbs
      - VIRTUAL_HOST=booking.srib.no
      - LETSENCRYPT_HOST=booking.srib.no
    volumes:
      - ./config/mrbs:/config
    ports:
      - 8888:80
    restart: always
    depends_on:
      - mrbs-db
    networks:
      - frontend
  mrbs-db:
    image: mariadb:latest
    container_name: mrbs_db
    environment:
      - PUID=1000
      - PGID=1000
      - MYSQL_ROOT_PASSWORD=kekekekek
      - TZ=Europe/Oslo
      - MYSQL_DATABASE=mrbs
      - MYSQL_USER=mrbs-user
      - MYSQL_PASSWORD=lolololol
    volumes:
      - ./config/mysql:/var/lib/mysql
    restart: always
    networks:
      - frontend
networks:
  frontend:
    external: true
```

Det er ikke noe spesielt triks med denne compose-filen, men det er greit å merke
seg volumene som importeres:

```yaml
volumes:
  - ./config/mrbs:/config
```

```yaml
volumes:
  - ./config/mysql:/var/lib/mysql
```

Altså, at `config`-mappen inneholder alle filene som database-beholderen og
mrbs-beholderen bruker.

### Endre variabler i mrbs

Om du ønsker å endre variabler i mrbs-tjenesten, for diverse grunner, kan du
gjøre det ved å manuelt endre php-koden. Da kan du bare åpne
`./config/mrbs/www/config.inc.php` med et tekstredigeringsprogram. Endringene
burde skje ganske umiddelbart.

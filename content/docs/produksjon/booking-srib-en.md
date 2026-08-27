+++
title = "booking.srib.no"
description = "Explanation of how the radio's room reservation system is set up"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/produksjon/booking-srib-no.md"
+++

### Short explanation

The room reservation service we provide for the radio is one of the most important ones. Without it,
it would not be possible for them to know which of their 200 active members
borrow the premises at which times.

The software is [Meeting Room Booking System](https://mrbs.sourceforge.io/), and
usually runs in a LAMP stack. In our case, we run it through Docker.

### Docker Compose file

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

There is no special trick to this Compose file, but it is useful to note
the volumes that are imported:

```yaml
volumes:
  - ./config/mrbs:/config
```

```yaml
volumes:
  - ./config/mysql:/var/lib/mysql
```

That is, the `config` folder contains all the files that the database container and
the mrbs container use.

### Change variables in mrbs

If you want to change variables in the mrbs service for various reasons, you can
do so by manually editing the PHP code. You can just open
`./config/mrbs/www/config.inc.php` in a text editor. The changes
should take effect almost immediately.

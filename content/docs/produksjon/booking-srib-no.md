+++
title = "booking.srib.no"
description = "Forklaring på hvordan romreservasjonssystemet til radioen er skrudd sammen"
date = 2024-01-23T08:00:00+00:00
updated = 2024-01-23T08:00:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

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
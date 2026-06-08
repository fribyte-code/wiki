+++
title = "skjema.srib.no"
description = "Explanation of how the radio's survey page is set up"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/produksjon/skjema-srib-no.md"
+++

### Short explanation

The student radio uses this service to create surveys. It is software called
[LimeSurvey](https://github.com/LimeSurvey/LimeSurvey).

### Docker Compose file

```yaml
version: "3"

services:
  limesurvey:
    image: docker.io/martialblog/limesurvey:latest
    restart: always
    networks:
      - frontend
    environment:
      - DB_TYPE=pgsql
      - DB_PORT=5432
      - DB_HOST=limesurvey-db
      - DB_PASSWORD=lololol
      - DB_NAME=limesurvey
      - DB_USERNAME=limesurvey
      - ADMIN_USER=admin
      - ADMIN_NAME=friByte
      - ADMIN_PASSWORD=kekekek
      - ADMIN_EMAIL=admin@fribyte.no
      - VIRTUAL_HOST=skjema.srib.no
      - LETSENCRYPT_HOST=skjema.srib.no
      - PUBLIC_URL=skjema.srib.no
      - VIRTUAL_PORT=8080
    volumes:
      - limesurvey:/var/www/html/upload/surveys
    depends_on:
      - limesurvey-db

  limesurvey-db:
    image: docker.io/postgres:10-alpine
    restart: always
    volumes:
      - limesurvey-db-data:/var/lib/postgresql
    environment:
      - POSTGRES_USER=limesurvey
      - POSTGRES_DB=limesurvey
      - POSTGRES_PASSWORD=lolololol
    networks:
      - frontend

networks:
  frontend:
    external: true

volumes:
  limesurvey:
  limesurvey-db-data:
```

+++
title = "wiki.srib.no"
description = "Forklaring på hvordan wiki-siden er skrudd sammen"
date = 2024-01-25T08:00:00+00:00
updated = 2024-01-25T08:00:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

### Kort forklaring

Wiki-siden til radioen er statisk nettside, laget med [Docsy-temaet til Hugo](https://github.com/google/docsy). Du kan finne [repoet til koden her](https://github.com/srib-dev/wiki). Repoet er egentlig bare klonet fra det originale repoet og kjører i en docker-beholder.

### Docker compose fil

```yaml
version: "3.3"

services:
  site:
    image: docsy/docsy-example
    restart: always
    build:
      context: .
    command: server
    ports:
      - "1313:1313"
    volumes:
      - .:/src
    environment:
      - VIRTUAL_HOST=wiki.srib.no
      - LETSENCRYPT_HOST=wiki.srib.no
      - VIRTUAL_PORT=1313
    networks:
      - frontend

networks:
 frontend:
  external: true
```
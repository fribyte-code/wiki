+++
title = "wiki.srib.no"
description = "Explanation of how the wiki site is set up"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/produksjon/wiki-srib-no.md"
+++

### Short explanation

The radio's wiki site is a static website built with
[Hugo's Docsy theme](https://github.com/google/docsy). You can find
[the code repository here](https://github.com/srib-dev/wiki). The repository is
really just cloned from the original repository and runs in a Docker container.

### Docker Compose file

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

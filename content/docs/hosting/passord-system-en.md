+++
title = "Passwords - Bitwarden"
description = "How we keep track of passwords"
template = "docs/page.html"
sort_by = "weight"
weight = 10002
draft = false

[extra]
lang = "en"
translation = "docs/hosting/passord-system.md"
+++

## Bitwarden password hosting

[https://bitwarden.fribyte.no/](https://bitwarden.fribyte.no/)

We keep track of all shared passwords for various services in our self-hosted
Bitwarden instance.

It runs as a Docker container on Pengebingen and is started with the following
docker-compose file located in Pengebingen's home directory.

```sh
version: '3'
services:
 vaultwarden:
  container_name: vaultwarden
  image: vaultwarden/server:latest
  restart: always
  volumes:
   - /vw-data/:/data/
  environment:
   - VIRTUAL_HOST=bitwarden.fribyte.no
   - LETSENCRYPT_HOST=bitwarden.fribyte.no
   - SIGNUPS_DOMAINS_WHITELIST=fribyte.no,fribyte.uib.no
   - ADMIN_TOKEN=SUPER-DUPER-SECRET-TOKEN
  network_mode: bridge
```

- This should start every time Pengebingen boots, but if you want to change
  settings and apply them, you can run
  `sudo docker-compose -f vaultwarden-compose.yaml up -d`. That will recreate
  the container and apply all settings.

### Backup

We currently do not have a backup of Pengebingen, and we need to sort that out,
especially the Bitwarden passwords. On Pengebingen, the most important things
to back up are:

- Docker volumes
- The Postgres database

If we have backups of the Docker volumes, we can fairly easily start everything
up again on another machine by restarting the Docker containers with the same
settings.

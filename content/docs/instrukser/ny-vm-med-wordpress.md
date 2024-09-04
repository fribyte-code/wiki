+++
title = "Ny VM med Wordpress"
description = "Ny VM med Wordpress"
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

If this VM is meant to have High Availability, please see [here](../ha-setup)

Følgende instruks passer for å sette opp en helt fresh VM med Wordpress med
automatisk letsencrypt.

1. Følg [/docs/instrukser/ny-vm](/docs/instrukser/ny-vm/) for å sette opp en ny
   vm.
   - Du trenger rundt 6-10GB
2. Sett opp domene [/docs/instrukser/domener](/docs/instrukser/domener/)
3. Installer docker:
   ```sh
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh ./get-docker.sh
   ```
4. Lag en fil `docker-compose.yml` med følgende innhold og bytt ut
   `<DOMENE-NAVN>` med faktisk domenenavn:

```yaml
version: "3.0"
services:
  nginx-proxy:
    image: jwilder/nginx-proxy
    container_name: nginx-proxy
    restart: always
    ports:
      - "80:80"
      - "443:443"
    expose:
      - 80
      - 443
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - /home/fribyte/nginx-proxy/nginx/certs:/etc/nginx/certs
      - /home/fribyte/nginx-proxy/nginx/vhost.d:/etc/nginx/vhost.d
      - /home/fribyte/nginx-proxy/nginx/html:/usr/share/nginx/html
      - /home/fribyte/nginx-proxy/nginx/dhparam:/etc/nginx/dhparam
    environment:
      DEFAULT_HOST: default.vhost

  letsencrypt:
    image: jrcs/letsencrypt-nginx-proxy-companion
    container_name: nginx-proxy-le
    restart: always
    environment:
      NGINX_PROXY_CONTAINER: nginx-proxy
    depends_on:
      - nginx-proxy
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /home/fribyte/nginx-proxy/nginx/certs:/etc/nginx/certs
      - /home/fribyte/nginx-proxy/nginx/vhost.d:/etc/nginx/vhost.d
      - /home/fribyte/nginx-proxy/nginx/html:/usr/share/nginx/html
      - /home/fribyte/nginx-proxy/nginx/dhparam:/etc/nginx/dhparam

  db:
    image: mariadb:latest
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
      - letsencrypt
    image: wordpress:latest
    volumes:
      - wp_content:/var/www/html/wp-content
    expose:
      - 80
    restart: always
    environment:
      VIRTUAL_HOST: <DOMENE-NAVN>
      LETSENCRYPT_HOST: <DOMENE-NAVN>
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    stdin_open: true
    tty: true

volumes:
  db_data:
  wp_content:
```

5. Kjør så `sudo docker compose up -d` for å starte opp docker-compose
6. Din nye wordpress side skal nå være tilgjengelig på https://DOMENE-NAVN.no/

### Øke allowed upload file size:

nginx-proxy har som default en ganske lav begrensing på opplasting av filer. Så
da kan man oppleve å få feilmeldingen `Request Entity Too Large`. For å fikse
dette må man legge til `client_max_body_size 25m;` i bunn av
`/home/fribyte/nginx-proxy/nginx/vhost.d`.

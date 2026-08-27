+++
title = "New VM with WordPress"
description = "New VM with WordPress"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/instrukser/ny-vm-med-wordpress.md"
+++

If this VM is meant to have High Availability, please see [here](../ha-setup)

The following instructions are suitable for setting up a completely fresh VM with
WordPress and automatic Let's Encrypt.

1. Follow [/docs/instrukser/ny-vm](/docs/instrukser/ny-vm/) to set up a new VM.
   - You need around 6-10GB
2. Set up the domain [/docs/instrukser/domener](/docs/instrukser/domener/)
3. Install Docker:
   ```sh
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh ./get-docker.sh
   ```
4. Create a `docker-compose.yml` file with the following content and replace
   `<DOMENE-NAVN>` with the actual domain name:

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

5. Then run `sudo docker compose up -d` to start docker-compose
6. Your new WordPress site should now be available at https://DOMENE-NAVN.no/

### Increase the allowed upload file size:

By default, `nginx-proxy` has a fairly low limit for file uploads. You may
therefore encounter the error message `Request Entity Too Large`. To fix this,
add `client_max_body_size 25m;` at the bottom of
`/home/fribyte/nginx-proxy/nginx/vhost.d`.

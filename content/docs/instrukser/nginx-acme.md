+++
title = "Nginx acme"
description = "Oppsett av nginx med acme modulen"
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

Følg installasjonstegene for nginx sitt offisielle repo på Ubuntu. Dette inkluderer ekstra moduler som ngx_http_acme_module.

https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#prebuilt_ubuntu

Installer ngx_http_acme_module nginx modulen

```bash
sudo apt install nginx-module-acme
```

Endre konfigurasjonsfilen til nginx

```bash
sudo nano /etc/nginx/nginx.conf
```

```conf
load_module modules/ngx_http_acme_module.so;

user fribyte;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    resolver 1.1.1.1;

    acme_issuer letsencrypt {
        uri         https://acme-v02.api.letsencrypt.org/directory;
        # contact   admin@example.test;
        state_path  /var/cache/nginx/acme-letsencrypt;

        accept_terms_of_service;
    }

    include /etc/nginx/conf.d/*.conf;
}
```

Skru av nginx sin default landingsside

```bash
sudo mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.disabled
```

Vi kan nå legge til en ny konfigurasjonsfil for siden vår i conf.d mappa. Her serverer vi en statisk side, men det er også mulig å bruke nginx som en revers proxy om ønsket.

```bash
sudo nano /etc/nginx/conf.d/example.conf
```

```conf
server {
    # Listener on port 80 is required to process ACME HTTP-01 challenges
    listen 80;
    listen [::]:80;

    server_name example.fribyte.no;

    # Redirect to https
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    listen [::]:443;

    server_name example.fribyte.no;

    # SSL
    acme_certificate letsencrypt;
    ssl_certificate       $acme_certificate;
    ssl_certificate_key   $acme_certificate_key;
    ssl_certificate_cache max=2;

    root /home/fribyte/example;

    index index.html;
    error_page 404 404.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Sjekk at nginx konfigurasjonen er a-ok

```
sudo nginx -t
```

Lag en folder i hjemmemappen til brukeren som nginx kan servere filene fra, filene kan plasseres hit av CI/CD.

```bash
mkdir ~/example
```

Profit
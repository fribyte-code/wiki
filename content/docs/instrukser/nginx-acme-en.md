+++
title = "Nginx ACME"
description = "Configuring nginx with the acme module"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/instrukser/nginx-acme.md"
+++

# Installation

Follow the installation steps for nginx's official repository on Ubuntu. This
contains extra modules such as `ngx_http_acme_module`.

https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#prebuilt_ubuntu

Install the `ngx_http_acme_module` nginx module

```bash
sudo apt install nginx-module-acme
```

# Configuration

Edit the nginx configuration file

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

Disable nginx's default landing page

```bash
sudo mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.disabled
```

We can now add a new configuration file for the site you want to serve in the
`conf.d` folder. In this example we serve a static site, but it is also
possible to use nginx as a reverse proxy if desired.

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

Check that the nginx configuration is OK

```
sudo nginx -t
```

Create a folder in the user's home directory that nginx can serve files from;
the files can then be copied here by a CI/CD pipeline or similar.

```bash
mkdir ~/example
```

Profit

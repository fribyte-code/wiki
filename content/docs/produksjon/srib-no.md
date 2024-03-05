+++
title = "srib.no"
description = "Studentradioen sin Wordpress hjemmeside"
date = 2024-01-11T14:00:00+00:00
updated = 2024-01-11T14:00:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

## Kort forklaring

[Studentradioen i Bergen](/docs/kunder/studentradioen) sin hjemmeside er en Wordpress-installasjon som kjører i et Docker-miljø. Docker-miljøet kjører på en virtuell maskin, som heter "Pengebingen", i [Proxmox-miljøet](/docs/maskiner/bolivar-skaftetrynet-pluto-cluster) vårt. 

## Docker compose fil

For å lansere nettsiden bruker vi docker compose. Det tillater oss å lagre konfigurasjonen mer sikkert over lengre tid, samt feilsøke og lansere mye enklere. 

```yaml
version: "3.0"
services:
  srib_wordpress:
    image: webdevops/php-apache:7.4-alpine
    container_name: srib-wp
    volumes:
    - /home/fribyte/srib.no/srib.no/hovedside/:/app/
    - wp_uploads:/app/hovedside/wp-content/uploads
    expose:
    - 80
    restart: always
    environment:
      VIRTUAL_PORT: 80
      VIRTUAL_HOST: srib.no
      LETSENCRYPT_HOST: srib.no
      PHP_OPCACHE_MEMORY_CONSUMPTION: 192
      PHP_OPCACHE_MAX_ACCELERATED_FILES: 10000
      PHP_OPCACHE_VALIDATE_TIMESTAMPS: 1
      PHP_OPCACHE_REVALIDATE_FREQ: 0
      PHP_OPCACHE_INTERNED_STRINGS_BUFFER: 16
      PHP_DISMOD: sodium,exif,pgsql,ioncube,amqp,bcmath,ldap,pcntl,redis,mongodb,xsl,sysvshm,sysvmsg,sysvsem
    stdin_open: true
    tty: true
    networks:
      - frontend

  db:
    image: mysql
    container_name: wp-mysql
    restart: always
    cap_add:
      - SYS_NICE
    environment:
      MYSQL_DATABASE: 'lolololol'
      MYSQL_USER: 'lolololol'
      MYSQL_PASSWORD: 'lolol'
      MYSQL_ROOT_PASSWORD: 'lolol'
      MYSQL_ALLOW_EMPTY_PASSWORD: 'false'
    ports:
      - '3306:3306'
    expose:
      - '3306'
    volumes:
      - wp_mysql:/var/lib/mysql
    networks:
      - frontend

volumes:
  wp_mysql:
  wp_uploads:
    driver: vieux/sshfs:latest
    driver_opts:
      sshcmd: root@158.37.6.43:/dalar/websites/srib/uploads/
      password: lolololol
      allow_other: ""

networks:
  frontend:
    external: true
```

NB: Det er viktig å bemerke seg `image: webdevops/php-apache:7.4-alpine`. Det er dette som bestemmer php-versjonen som wordpress-installasjonen skal bruke. Temaet som nettsiden bruker, er ganske avhengig av at riktig php-versjon. Hvis ikke, kræsjer hele nettsiden. Om denne versjonen, 7.4, blir for utdatert for Wordpress-utvidelsene, må versjonen oppdateres. 

## Filopplasting og fillagring

Denne blokken:

```yaml
volumes:
  # ....
  wp_uploads:
    driver: vieux/sshfs:latest
    driver_opts:
      sshcmd: root@158.37.6.43:/dalar/websites/srib/uploads/
      password: lolololol
      allow_other: ""
```

Er det som tillater lagring av opplastede media på [fillagringsmaskinen, Skrue](/docs/maskiner/skrue). Det er nå kofigurert slik at når en laster opp filer til Wordpress-siden, gjennom brukergrensesnittet, lagres filene på Skrue, i den designerte mappen i `sshcmd`. 

For å kunne gjøre dette, må du installere Docker-utvidelsen `vieux/sshfs:latest`. Det gjør du slik: `docker plugin install vieux/sshfs`. `password` blir da passordet til brukeren du logger på maskinen som. I dette tilfellet er `lolololol` passordet til `root`.

## Database-lagring

Wordpress-installasjonen trenger naturligvis en database. Det vi bruker som database er en mysql-server på en egen [database-VM](/docs/instrukser/migrere-database).

Den er koblet opp ved å spesifisere følgende i `wp-config.php`:

```php

#.........

define('DB_NAME', 'db_name');

/** MySQL database username */
define('DB_USER', 'db_user');

/** MySQL database password */
define('DB_PASSWORD', 'lolololol');

/** MySQL hostname */
define('DB_HOST', '158.37.6.37');

#.........
```

NB: `db_name`, `db_user` og `lolololol` er bare selvfølgelig plassholdere. 

For at denne tilkoblingen skal fungere, må du åpne for tilkobling i mysql-tjeneren, samt gi de riktige rettighetene i sql-databasen til `db_user`.
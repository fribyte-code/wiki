+++
title = "srib.no"
description = "The Student Radio's WordPress website"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/produksjon/srib-no.md"
+++

## Brief explanation

[Studentradioen i Bergen](/docs/kunder/studentradioen)'s website is a
WordPress installation that runs in a Docker environment. The Docker
environment runs on a virtual machine called "Pengebingen" in our
[Proxmox environment](/docs/maskiner/bolivar-skaftetrynet-pluto-cluster).

## Docker compose file

To launch the website, we use docker compose. It allows us to store the
configuration more safely over time, as well as troubleshoot and launch much
more easily.

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
      MYSQL_DATABASE: "lolololol"
      MYSQL_USER: "lolololol"
      MYSQL_PASSWORD: "lolol"
      MYSQL_ROOT_PASSWORD: "lolol"
      MYSQL_ALLOW_EMPTY_PASSWORD: "false"
    ports:
      - "3306:3306"
    expose:
      - "3306"
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

Note: it is important to note `image: webdevops/php-apache:7.4-alpine`. This is
what determines the PHP version that the WordPress installation will use. The
theme used by the website is quite dependent on the correct PHP version.
Otherwise, the entire website crashes. If this version, 7.4, becomes too
outdated for the WordPress extensions, the version must be updated.

## File upload and file storage

This block:

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

Is what allows uploaded media to be stored on
[the file storage machine, Skrue](/docs/maskiner/skrue). It is now configured so
that when you upload files to the WordPress site through the user interface, the
files are stored on Skrue, in the designated folder in `sshcmd`.

To be able to do this, you must install the Docker plugin
`vieux/sshfs:latest`. You do that like this: `docker plugin install vieux/sshfs`.
`password` then becomes the password of the user you log in to the machine as.
In this case, `lolololol` is the password for `root`.

## Database storage

The WordPress installation naturally needs a database. What we use as the
database is a MySQL server on a separate
[database VM](/docs/instrukser/migrere-database).

It is connected by specifying the following in `wp-config.php`:

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

Note: `db_name`, `db_user` and `lolololol` are of course only placeholders.

For this connection to work, you must allow connections in the MySQL server, as
well as grant the correct permissions in the SQL database to `db_user`.

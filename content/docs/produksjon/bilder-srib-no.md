+++
title = "bilder.srib.no"
description = "Forklaring på hvordan bildelagringstjenesten til studentradioen er rigget opp"
date = 2024-02-10T08:00:00+00:00
updated = 2024-02-10T08:00:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++


### Kort forklaring

Dette er en tjeneste som studentradioen "bruker" for å lagre bildene sine. Det er en tjeneste som heter [PhotoPrism](https://github.com/photoprism/photoprism). 

### Docker compose fil

```yaml
version: '3.5'

# Example Docker Compose config file for PhotoPrism (Linux / AMD64)
#
# Note:
# - Running PhotoPrism on a server with less than 4 GB of swap space or setting a memory/swap limit can cause unexpected
#   restarts ("crashes"), for example, when the indexer temporarily needs more memory to process large files.
# - If you install PhotoPrism on a public server outside your home network, please always run it behind a secure
#   HTTPS reverse proxy such as Traefik or Caddy. Your files and passwords will otherwise be transmitted
#   in clear text and can be intercepted by anyone, including your provider, hackers, and governments:
#   https://docs.photoprism.app/getting-started/proxies/traefik/
#
# Documentation : https://docs.photoprism.app/getting-started/docker-compose/
# Docker Hub URL: https://hub.docker.com/r/photoprism/photoprism/
#
# DOCKER COMPOSE COMMAND REFERENCE
# see https://docs.photoprism.app/getting-started/docker-compose/#command-line-interface
# --------------------------------------------------------------------------
# Start    | docker-compose up -d
# Stop     | docker-compose stop
# Update   | docker-compose pull
# Logs     | docker-compose logs --tail=25 -f
# Terminal | docker-compose exec photoprism bash
# Help     | docker-compose exec photoprism photoprism help
# Config   | docker-compose exec photoprism photoprism config
# Reset    | docker-compose exec photoprism photoprism reset
# Backup   | docker-compose exec photoprism photoprism backup -a -i
# Restore  | docker-compose exec photoprism photoprism restore -a -i
# Index    | docker-compose exec photoprism photoprism index
# Reindex  | docker-compose exec photoprism photoprism index -f
# Import   | docker-compose exec photoprism photoprism import
#
# To search originals for faces without a complete rescan:
# docker-compose exec photoprism photoprism faces index
#
# All commands may have to be prefixed with "sudo" when not running as root.
# This will point the home directory shortcut ~ to /root in volume mounts.

services:
  photoprism:
    ## Use photoprism/photoprism:preview for testing preview builds:
    image: photoprism/photoprism:latest
    container_name: photoprism
    depends_on:
      - mariadb
    ## Don't enable automatic restarts until PhotoPrism has been properly configured and tested!
    ## If the service gets stuck in a restart loop, this points to a memory, filesystem, network, or database issue:
    ## https://docs.photoprism.app/getting-started/troubleshooting/#fatal-server-errors
    restart: always
    security_opt:
      - seccomp:unconfined
      - apparmor:unconfined
    ports:
      - "2342:2342" # HTTP port (host:container)
    environment:
      PHOTOPRISM_ADMIN_PASSWORD: "kekekek"  # YOUR INITIAL ADMIN PASSWORD (MINIMUM 8 CHARACTERS, USERNAME "admin")
      PHOTOPRISM_SITE_URL: "https://bilder.srib.no/"  # public server URL incl http:// or https:// and /path, :port(2342) is optional
      PHOTOPRISM_ORIGINALS_LIMIT: 50000               # file size limit for originals in MB (increase for high-res video)
      PHOTOPRISM_HTTP_COMPRESSION: "gzip"            # improves transfer speed and bandwidth utilization (none or gzip)
      PHOTOPRISM_LOG_LEVEL: "info"                   # log level: trace, debug, info, warning, error, fatal, or panic
      PHOTOPRISM_PUBLIC: "false"                     # no authentication required (disables password protection)
      PHOTOPRISM_READONLY: "false"                   # do not modify originals directory (reduced functionality)
      PHOTOPRISM_EXPERIMENTAL: "false"               # enables experimental features
      PHOTOPRISM_DISABLE_CHOWN: "false"              # disables storage permission updates on startup
      PHOTOPRISM_DISABLE_WEBDAV: "false"             # disables built-in WebDAV server
      PHOTOPRISM_DISABLE_SETTINGS: "false"           # disables settings UI and API
      PHOTOPRISM_DISABLE_TENSORFLOW: "true"         # disables all features depending on TensorFlow
      PHOTOPRISM_DISABLE_FACES: "true"              # disables facial recognition
      PHOTOPRISM_DISABLE_CLASSIFICATION: "false"     # disables image classification
      PHOTOPRISM_DISABLE_RAW: "false"                # disables indexing and conversion of RAW files
      PHOTOPRISM_RAW_PRESETS: "false"                # enables applying user presets when converting RAW files (reduces performance)
      PHOTOPRISM_JPEG_QUALITY: 85                    # image quality, a higher value reduces compression (25-100)
      PHOTOPRISM_DETECT_NSFW: "false"                # flag photos as private that MAY be offensive (requires TensorFlow)
      PHOTOPRISM_UPLOAD_NSFW: "true"                 # allows uploads that MAY be offensive
      # PHOTOPRISM_DATABASE_DRIVER: "sqlite"         # SQLite is an embedded database that doesn't require a server
      PHOTOPRISM_DATABASE_DRIVER: "mysql"            # use MariaDB 10.5+ or MySQL 8+ instead of SQLite for improved performance
      PHOTOPRISM_DATABASE_SERVER: "mariadb:3306"     # MariaDB or MySQL database server (hostname:port)
      PHOTOPRISM_DATABASE_NAME: "photoprism_db"         # MariaDB or MySQL database schema name
      PHOTOPRISM_DATABASE_USER: "photoprism"         # MariaDB or MySQL database user name
      PHOTOPRISM_DATABASE_PASSWORD: "lololol"       # MariaDB or MySQL database user password
      PHOTOPRISM_SITE_CAPTION: "Studentradioen i Bergen"
      PHOTOPRISM_SITE_DESCRIPTION: "Bildene til SRiB"    # meta site description
      PHOTOPRISM_SITE_AUTHOR: "SRiB"                         # meta site author
      #PHOTOPRISM_DEBUG: "true"
      ## Run/install on first startup (options: update, gpu, tensorflow, davfs, clitools, clean):
      # PHOTOPRISM_INIT: "gpu tensorflow"
      ## Hardware Video Transcoding (for sponsors only due to high maintenance and support costs):
      # PHOTOPRISM_FFMPEG_ENCODER: "software"        # FFmpeg encoder ("software", "intel", "nvidia", "apple", "raspberry")
      # PHOTOPRISM_FFMPEG_BITRATE: "32"              # FFmpeg encoding bitrate limit in Mbit/s (default: 50)
      ## Switch to a non-root user after initialization (supported IDs are 33, 50-99, 500-600, and 900-1200):
      # PHOTOPRISM_UID: 1000
      # PHOTOPRISM_GID: 1000
      # PHOTOPRISM_UMASK: 0000
      VIRTUAL_HOST: "bilder.srib.no"
      LETSENCRYPT_HOST: "bilder.srib.no"
      VIRTUAL_PORT: 2342
    ## Start as a non-root user before initialization (supported IDs are 33, 50-99, 500-600, and 900-1200):
    # user: "1000:1000"
    ## Share hardware devices with FFmpeg and TensorFlow (optional):
    # devices:
    #  - "/dev/dri:/dev/dri"                         # Intel QSV
    #  - "/dev/nvidia0:/dev/nvidia0"                 # Nvidia CUDA
    #  - "/dev/nvidiactl:/dev/nvidiactl"
    #  - "/dev/nvidia-modeset:/dev/nvidia-modeset"
    #  - "/dev/nvidia-nvswitchctl:/dev/nvidia-nvswitchctl"
    #  - "/dev/nvidia-uvm:/dev/nvidia-uvm"
    #  - "/dev/nvidia-uvm-tools:/dev/nvidia-uvm-tools"
    #  - "/dev/video11:/dev/video11"                 # Raspberry V4L2
    working_dir: "/photoprism" # do not change or remove
    ## Storage Folders: "~" is a shortcut for your home directory, "." for the current directory
    volumes:
      # "/host/folder:/photoprism/folder"                # Example
      - "/home/fribyte/photoprism/srib-bilder:/photoprism/originals"               # Original media files (DO NOT REMOVE
      # - "/example/family:/photoprism/originals/family" # *Additional* media folders can be mounted like this
      # - "~/Import:/photoprism/import"                  # *Optional* base folder from which files can be imported to originals
      - "./storage:/photoprism/storage"                  # *Writable* storage folder for cache, database, and sidecar files (DO NOT REMOVE) 
      - "/home/fribyte/photoprism/srib-bilder"
    networks:
      - frontend
  ## Database Server (recommended)
  ## see https://docs.photoprism.app/getting-started/faq/#should-i-use-sqlite-mariadb-or-mysql
  mariadb:
    ## If MariaDB gets stuck in a restart loop, this points to a memory or filesystem issue:
    ## https://docs.photoprism.app/getting-started/troubleshooting/#fatal-server-errors
    restart: always
    image: mariadb:10.7
    container_name: mariadb
    security_opt:
      - seccomp:unconfined
      - apparmor:unconfined
    command: mysqld --innodb-buffer-pool-size=128M --transaction-isolation=READ-COMMITTED --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --max-connections=512 --innodb-rollback-on-timeout=OFF --innodb-lock-wait-timeout=120
    ## Never store database files on an unreliable device such as a USB flash drive, an SD card, or a shared network folder:
    volumes:
      - "./database:/var/lib/mysql" # DO NOT REMOVE
    environment:
      MARIADB_AUTO_UPGRADE: "1"
      MARIADB_INITDB_SKIP_TZINFO: "1"
      MARIADB_DATABASE: "photoprism_db"
      MARIADB_USER: "photoprism"
      MARIADB_PASSWORD: "lololol"
      MARIADB_ROOT_PASSWORD: "jajajaja"
    networks:
      - frontend

  ## Watchtower upgrades services automatically (optional)
  ## see https://docs.photoprism.app/getting-started/updates/#watchtower
  #
  # watchtower:
  #   restart: unless-stopped
  #   image: containrrr/watchtower
  #   environment:
  #     WATCHTOWER_CLEANUP: "true"
  #     WATCHTOWER_POLL_INTERVAL: 7200 # checks for updates every two hours
  #   volumes:
  #     - "/var/run/docker.sock:/var/run/docker.sock"
  #     - "~/.docker/config.json:/config.json" # optional, for authentication if you have a Docker Hub account
  #
networks:
 frontend:
  external: true
```

Det som er greit å bemerke seg her, er dette:

```yaml
    volumes:
      (...)

      - "/home/fribyte/photoprism/srib-bilder:/photoprism/originals"
      
      (...)

      - "/home/fribyte/photoprism/srib-bilder"
```

Vi importerer en lokal mappe på VM-en inn i Docker-beholderen. Dette er samme praksis som [podcast.srib.no](/docs/produksjon/podcast-srib-no)

Den lokale mappen er også en importert mappe fra NAS-et til Studentradioen. Vi bruker den følgende linjen i fstab for å importere mappen fra NAS-et. 

```
//158.37.6.118/Bilder/  /home/fribyte/photoprism/srib-bilder    cifs    vers=3.0,username=fribyte,password=aiaiaiai,noexec  0 0
```
Brukernavn og passord, er til brukeren i NAS-et til studentradioen. Fildelingen krever at studentradioen godkjenner at vi kan koble oss til NAS-et deres, gjennom programvaren til NAS-et deres. Passordet kan du finne i [bitwarden.fribyte.no](https://bitwarden.fribyte.no)

Dette er for både lesing og skriving, selvfølgelig. Filer som ligger i den mappen på NAS-et blir vist i `bilder.srib.no`, og alle bilder som blir lastet opp til `bilder.srib.no`, ender opp på NAS-et deres.

NAS-et deres er av typen QNAP TS-853BU, [og du kan lese mer om det her](https://wiki.srib.no/docs/machines/servers/mimir/).


### client_max_body_size problemer

I utgangspunktet var det et problem med at størrelsen på bildet som ble forsøkt lastet opp var for stort. Dette var fordi `client_max_body_size` i nginx-proxy sin nginx.conf var satt 1MB, mens Azuracast sin `client_max_body_size` var satt til 50MB. Så da satte vi nginx-proxy sin `client_max_body_size` til 50MB også. 
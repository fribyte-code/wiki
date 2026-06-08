+++
title = "Castopod"
description = "Open-source podcast server"
template = "docs/page.html"
sort_by = "weight"
weight = 10002
draft = false

[extra]
lang = "en"
translation = "docs/hosting/castopod.md"
+++

friByte runs its own podcast server to distribute its own podcast,
"Teknisk Tenketank". For this purpose we use
[Castopod](https://docs.castopod.org/) and it is available at
[podcast.fribyte.no](https://podcast.fribyte.no).

### Docker Compose file

I used a Docker Compose file based on
[the one Castopod provides by default](https://docs.castopod.org/getting-started/docker.html#example-usage).
I naturally changed it so that it fits our environment.

Here is the Docker Compose file that ended up working:

```yaml
version: "3.7"

services:
  app:
    image: castopod/app:latest
    container_name: "castopod-app"
    volumes:
      - castopod-media:/opt/castopod/public/media
      - ttt_mount:/opt/castopod/public/media/podcasts/teknisktenketank

    environment:
      MYSQL_HOST: cpmariadb
      MYSQL_DATABASE: castopod
      MYSQL_USER: castopod
      MYSQL_PASSWORD: lelelelelel
      CP_BASEURL: "http://podcast.fribyte.no"
      CP_ANALYTICS_SALT: enellerannenstreng # generated with: tr -dc \!\#-\&\(-\[\]-\_a-\~ </dev/urandom | head -c 64
      CP_CACHE_HANDLER: redis
      CP_REDIS_HOST: redis
      CP_DATABASE_HOSTNAME: castopod-mariadb
      VIRTUAL_HOST: podcast.fribyte.no
      LETSENCRYPT_HOST: podcast.fribyte.no
    networks:
      - proxy_network
      - castopod-db
    restart: always

  web-server:
    image: castopod/web-server:latest
    container_name: "castopod-web-server"
    volumes:
      - castopod-media:/var/www/html/media
      - ttt_mount:/var/www/html/media/podcasts/teknisktenketank

    networks:
      - proxy_network
    ports:
      - 8080:80
    restart: always
    environment:
      VIRTUAL_HOST: podcast.fribyte.no
      LETSENCRYPT_HOST: podcast.fribyte.no

  cpmariadb:
    image: mariadb:10.5
    container_name: "castopod-mariadb"
    networks:
      - castopod-db
    volumes:
      - castopod-db:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: lolololol
      MYSQL_DATABASE: castopod
      MYSQL_USER: castopod
      MYSQL_PASSWORD: lelelelelel
    restart: always

  redis:
    image: redis:7.0-alpine
    container_name: "castopod-redis"
    volumes:
      - castopod-cache:/data
    networks:
      - proxy_network

  # this container is optional
  # add this if you want to use the video clips feature
#  video-clipper:
#    image: castopod/video-clipper:latest
#    container_name: "castopod-video-clipper"
#    volumes:
#      - castopod-media:/opt/castopod/public/media
#    environment:
#      MYSQL_DATABASE: castopod
#      MYSQL_USER: castopod
#      MYSQL_PASSWORD: lelelelelel
#    networks:
#      - proxy-network
#    restart: always

volumes:
  castopod-media:
  castopod-db:
  castopod-cache:
  ttt_mount:
    driver: vieux/sshfs:latest
    driver_opts:
      sshcmd: root@158.37.6.43:/dalar/podcast_ttt
      password: kekekekek
      allow_other: ""

networks:
  proxy_network:
    external: true
  castopod-db:
```

`ttt_mount` is a Docker volume that imports files stored on Skrue
(explained in more detail a bit further down). As you can see in the volume
definition, SSHFS is used to mount the files. The password in this
configuration becomes the SSH password for the `root` user on Konrad.

You can also use an SSH key for authentication. That can be done like this:

```yaml
ttt_mount:
  driver: vieux/sshfs:latest
  driver_opts:
    sshcmd: root@158.37.6.43:/dalar/podcast_ttt
    IdentityFile: "/root/.ssh/ssh-key-file"
    allow_other: ""
```

But in that case it is actually necessary for the SSH key to be located in
`/root/.ssh/..`. Otherwise, the `vieux/sshfs` extension will not find the SSH
key.
[Source](https://github.com/vieux/docker-volume-sshfs/issues/58#issuecomment-447317661)

I installed the extension by running:
`docker plugin install --grant-all-permissions vieux/sshfs` in Pengebingen.

These files are mounted directly into the Docker containers `castopod-app` and
`castopod-web-server` from Skrue. It is important that the files are available
in the respective directory in **both** containers.

The podcast episodes created through the admin panel at podcast.fribyte.no are
created under `../media/podcasts/<podcast-name>`. Therefore, when an episode is
uploaded through the admin panel, it is stored on the Docker volume mounted
from Pengebingen into the containers.

### Importing audio files

To import the podcast episodes we had already published through Studentradioen i
Bergen, I used Castopod's built-in import service. It loaded all episodes, as
well as metadata, through the RSS feed
`https://podcast.srib.no/feed/254`.

### Troubleshooting

Initially, I got a `404` error in the browser when the website tried to load
the audio files and the accompanying image files. This was because I had only
mounted the volume into the `castopod-app` container, and not into the
`castopod-web-server` container.

I fixed this by mounting `ttt_mount` into `castopod-web-server` as well.

### Sources:

[Castopod: Getting Started - Official Docker images](https://docs.castopod.org/getting-started/docker.html#example-usage),
last accessed 2023-03-24.

[Mounting a SFTP (SSH) Share as a Volume in Docker-Compose](https://simplytim.io/mounting-a-sftp-ssh-share-as-a-volume-in-docker-compose/),
last accessed 2023-03-24.

[How do debug when password-less mount does not work](https://github.com/vieux/docker-volume-sshfs/issues/58#issuecomment-447317661),
last accessed 2023-03-24.

+++
title = "podcast.srib.no"
description = "Explanation of how the student radio's podcast service is set up"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/produksjon/podcast-srib-no.md"
+++

### The code

The code is a homegrown Django application, written by a former technical
manager at the student radio station.
[You can read more about the code here.](https://github.com/srib-dev/podkast.srib.no/blob/master/README.md)

### Docker Compose file

```yaml
version: "3"
services:
  srib-podcast:
    container_name: srib-podcast
    image: srib-podcast
    restart: always
    volumes:
      - type: bind
        source: /home/fribyte/srib-nas-mount/digasLydfiler/podcast
        target: /media/podcast
        read_only: true
    networks:
      - frontend
    environment:
      - VIRTUAL_HOST=podcast.srib.no
      - LETSENCRYPT_HOST=podcast.srib.no

networks:
  frontend:
    external: true
```

It is a relatively simple Compose file, but it is worth noting:

```yaml
volumes:
  - type: bind
    source: /home/fribyte/srib-nas-mount/digasLydfiler/podcast
    target: /media/podcast
    read_only: true
```

This block binds a directory on the student radio's local NAS,
`/home/fribyte/srib-nas-mount/digasLydfiler/podcast`, into the Docker container, in
the `/media/podcast` directory.

### File sharing

If it was not obvious, then
`/home/fribyte/srib-nas-mount/digasLydfiler/podcast` is a local directory on the VM.
The contents of that directory come from the student radio's local NAS, which is in
their server room, but on the same subnet.

We use CIFSv3 to mount the relevant directory from their NAS into
our VM, with this line:
`//158.37.6.118/NAS/ /home/fribyte/srib-nas-mount cifs vers=3.0,username=fribyte,password=kekekekek,noexec 0 0`
in `/etc/fstab`.

The username and password are for the user on the student radio's NAS. The file sharing
requires the student radio to approve that we can connect to their NAS,
through the software on their NAS. You can find the password in
[bitwarden.fribyte.no](https://bitwarden.fribyte.no)

Their NAS is a QNAP TS-853BU,
[and you can read more about it here](https://wiki.srib.no/docs/machines/servers/mimir/).

### Troubleshooting

The podcast service will crash if it cannot access the student radio's NAS.
This can happen because the NAS is down or due to network issues. This usually resolves
itself after a while, but it may be necessary to run `sudo mount -a` to mount
external fstab files again. Then all definitions in `/etc/fstab` will be
mounted again.

- To check whether the podcast container is running:

```sh
# 1. SSH inn på VM-en
ssh fribyte@andeby.s.fribyte.no
ssh root@skaftetrynet.s.fribyte.no
ssh fribyte@podcast.srib.no
# 2. Sjekk om containeren kjører
docker ps | grep pod # Om ingen linjer kommer opp, så kjører ikke containeren
# 3. Mount fstab på nytt bare for å være sikker
sudo mount -a
# 4. Start containeren
cd podcast
sudo docker-compose up -d
```

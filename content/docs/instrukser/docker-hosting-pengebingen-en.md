+++
title = "Docker Hosting Pengebingen"
description = "How to host Docker containers"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/instrukser/docker-hosting-pengebingen.md"
+++

## Pengebingen

[pengebingen.fribyte.no](https://pengebingen.fribyte.no) is a VM created to
host internal docker containers. The advantage of docker containers is that
they use far fewer resources than VMs and include everything needed without
having to install all sorts of strange packages to get the service working.

It is set up with letsencrypt renewal through acme-companion
[nginx-proxy-acme companion](https://github.com/nginx-proxy/acme-companion).

It has an automatic nginx reverse proxy!

- So the only thing needed to arrange an ssl certificate is to add
  `VIRTUAL_HOST=domain` and `LETSENCRYPT_HOST=doamin` as docker container env
  variables.

## Setting up a new website:

1. Create a docker image of the website you want to host
2. Set up a subdomain by following the documentation for
   [domains](@/docs/instrukser/domener.md). Set the domain so that it points to
   the IP address of pengebingen.fribyte.no.
3. Run the docker image on pengebingen as described on the acme-companion
   page:

```sh
sudo docker run --detach \
    --name {definer-pent-navn-her} \
    --restart always \
    --env "VIRTUAL_PORT=3000" \
    --env "VIRTUAL_HOST=subdomain.fribyte.no" \
    --env "LETSENCRYPT_HOST=subdomain.fribyte.no" \
    {docker-image}
```

- It is important that `--restart always` is included to ensure that the
  container starts on docker restart or VM reboot.
- Virtual_host can be omitted if the service in the docker image already runs
  on port 80

4. If the DNS record has become active, the service will be available at
   subdomain.fribyte.no with an SSL certificate already set up.

### Communication between docker containers:

If you need to talk to a database, for example postgres, you must connect the
container to the same docker network that postgres is running on. Today this is
`pengebingen-docker-network`. Then the startup of the docker container becomes,
for example:

```sh
sudo docker run --detach \
    --name fribyte-ctf \
    --restart always \
    --env "VIRTUAL_PORT=8080" \
    --env "VIRTUAL_HOST=ctf.fribyte.no" \
    --env "LETSENCRYPT_HOST=ctf.fribyte.no" \
    --env "ADMIN_PASSWORD=PASSORD" \
    --env "DB_POSTGRES_CONNECTION_STRING=postgres://postgres:postgres@postgres:5432/ctf" \
    --network pengebingen-docker-network \
    mathiash98/fribyte-ctf
```

- Remember that when communicating between docker containers you do not use
  `localhost`, but rather the hostname of each container. So the connection to
  postgres becomes `postgres:5432` instead of `localhost:5432`.

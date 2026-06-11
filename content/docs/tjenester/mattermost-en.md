+++
title = "Mattermost"
description = "Internal communication"
template = "docs/page.html"
sort_by = "weight"
weight = 10003

[extra]
lang = "en"
translation = "docs/tjenester/mattermost.md"
+++

For all internal communication, we use Mattermost.

## Mandatory channels

New members are automatically added to the channels below, but make sure you are
in them.

### `viktig`

Important messages about meetings, events, and similar topics are posted here,
so pay attention to it. We try to be good about using `@channel`.

### Other mandatory channels

```json
"ExperimentalDefaultChannels": [
  "generelt",
  "admis",
  "viktig",
  "ymse",
  "sosialt",
  "[drift] alt",
  "dugnad"
]
```

### Update the list

1. Run `mmctl config edit`
1. Find `ExperimentalDefaultChannels`
1. Add or remove the channel in the array.

## Channel names (naming convention)

- Operations-related channels should have the `[drift]` prefix.
- Channels that only cover Gitea updates should have the `[gitea]` prefix.
- Channels that cover status changes related to one or more servers should have
  the `[drift:status]` prefix.

If you are unsure, ask in `generelt`; we can always rename channels.

## Webhooks

Mattermost supports sending and receiving webhooks, and that is what we use to
get status updates from Gitea.

You can access these by clicking in the upper-left corner and then choosing
**Integrations**.

## Make configuration changes directly from your own machine

Mattermost provides a nice command-line tool,
[`mmctl`](https://docs.mattermost.com/manage/mmctl-command-line-tool.html).

1. Install `mmctl`
1. Generate your own token for your Mattermost user
   1. Click yourself in the upper-right corner
   1. Click **Profile**
   1. Choose **Security** from the sidebar
   1. Create a **token**, and save the **Access Token**.
1. Run
   `mmctl auth login https://chat.fribyte.no --name fribyte --access-token <your-own-token>`
1. Profit! You can now use `mmctl` locally to do various things.

## Renew LetsEncrypt certificate

Mattermost does currently **not have automatic certificate renewal**, so the
following steps need to be performed until we fix an automatic cron job or
install `nginx-acme-companion`.

Steps:

1. Create a backup of the `mattermost` VM
2. SSH into the VM after the backup is finished
3. Run the following commands

```sh
sudo docker run --rm --name certbot \
    --network mattermost \
    -v "/home/fribyte/docker/certs/etc/letsencrypt:/etc/letsencrypt" \
    -v "/home/fribyte/docker/certs/lib/letsencrypt:/var/lib/letsencrypt" \
    -v shared-webroot:/usr/share/nginx/html \
    certbot/certbot renew --webroot-path /usr/share/nginx/html
```

```sh
sudo docker-compose -f ./docker/docker-compose.yml -f ./docker/docker-compose.nginx.yml restart
```

## Alternative LetsEncrypt renewal process

First we need to stop the Mattermost container, as we need to take over port 80.

```sh
sudo docker-compose -f ./docker/docker-compose.yml -f ./docker/docker-compose.nginx.yml down
```

Then run the certbot container in standalone mode:

```sh
sudo docker run --rm --name certbot \
   -v "/home/fribyte/docker/certs/etc/letsencrypt:/etc/letsencrypt" \
   -v "/home/fribyte/docker/certs/lib/letsencrypt:/var/lib/letsencrypt" \
   -v shared-webroot:/usr/share/nginx/html -p 80:80 \
   certbot/certbot certonly --standalone -d chat.fribyte.no --agree-tos -m renew@fribyte.no
```

Then restart Mattermost:

```sh
sudo docker-compose -f ./docker/docker-compose.yml -f ./docker/docker-compose.nginx.yml up -d
```

### Create new certificate configs

If the steps above do not work, it is also possible to create new certificate
configuration files with the following command.

_For future generations: to issue a new certificate, the `nginx_mattermost`
container must be stopped. This is because the certbot container script needs to
take over port 80 temporarily in order to complete the SSL handshake. Source:
several hours of debugging._

```sh
bash scripts/issue-certificate.sh -d chat.fribyte.no -o ${PWD}/certs
```

Commands are fetched from:

- [https://docs.mattermost.com/install/install-docker.html](https://docs.mattermost.com/install/install-docker.html)
- [https://github.com/mattermost/docker/blob/main/docs/issuing-letsencrypt-certificate.md](https://github.com/mattermost/docker/blob/main/docs/issuing-letsencrypt-certificate.md)

4. Test that Mattermost still works by sending a message

## Things to be aware of

- The Mattermost VM runs on `pluto` (2026-06-11)
- Mattermost is installed with
  **[Mattermost Omnibus CLI](https://docs.mattermost.com/install/installing-mattermost-omnibus.html)**
  - Not all config fields in `/opt/mattermost/config.json` are editable that
    way (or they will not update). See
    [this link](https://docs.mattermost.com/install/installing-mattermost-omnibus.html)
    for more information.
  - Check `/etc/mattermost/mmomni.yml` to see whether there are fields that
    should be changed there before changing them in `config.json`.

## Update Mattermost

1. Fetch the latest version of Mattermost
   - `sudo docker pull mattermost/mattermost-enterprise-edition:latest`
2. Stop the Mattermost container
   - `sudo docker stop mattermost`
3. Check whether the `mattermost` container has stopped
   - `sudo docker ps -a`
4. Check whether the `mattermost` container is gone
   - `sudo docker rm mattermost`
5. Fetch the latest version of Mattermost again
   - `sudo docker pull mattermost/mattermost-enterprise-edition:latest`
6. Run the script `./start_mm.sh`

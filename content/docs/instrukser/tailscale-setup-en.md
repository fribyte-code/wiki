+++
title = "Tailscale Setup for a New VM"
description = "Connect a new VM to the Tailscale network"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/instrukser/tailscale-setup.md"
+++

# Join a new VM to our internal tailnet

## Getting a preauthenticated key for joining the node

1. Make sure you are connected to the tailnet already then ssh into the headscale VM

```bash
ssh fribyte@headscale
```

2. Create a new preauth key that is tagged with the server tag. This will create a temporary key you need to join the new node

```bash
sudo headscale preauthkeys create --tags tag:server
```
## Joining the node with the preauth key

1. Install tailscale using the following command

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

2. Join the node by supplying the preauthkey you generated in the last step by replacing `<preauthkey>` in the command

```bash
sudo tailscale up --ssh --accept-dns=false --login-server https://headscale.fribyte.no --auth-key <preauthkey>
```

3. Check that the VM is now accessible over tailscale by running

```
ssh fribyte@<VM-HOSTNAME>
```

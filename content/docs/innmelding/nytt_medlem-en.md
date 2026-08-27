+++
title = "New member"
description = "How to onboard a new member"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/innmelding/nytt_medlem.md"
+++

This is a guide for how to give new members access to our relevant systems.

## Migadu

Migadu is the email service we use internally.

You can log in to the admin panel by going to:
[admin.migadu.com/login](https://admin.migadu.com/public/login). You log in
using your own user credentials, but the user must have **administrator**
rights.

To create a new email address for the new member:

1. Log in with an admin user.
2. Go to `Email Domains` -> `friByte.no`.
3. Under `All Addresses` or `Mailboxes`, click `New Mailbox`.
4. Fill in the member's information.
5. Click `Create Mailbox`.
6. The new member can check their inbox at
   [webmail.migadu.com](https://webmail.migadu.com/) and add the email server
   to their own mail app. Instructions are available at
   [migadu.com/guides](https://www.migadu.com/guides/).
7. Profit.

### Add the new member to the mailing lists

1. Log in with an admin user.
2. Go to `Mailboxes`.
3. Go to `List` -> `aktive@friByte.no`.
4. Under `Overview` -> `Delegation`.
5. Add the new email address in the text field for `Local Recipients`.
6. Save the changes.
7. Even more profit.

## Mattermost

You can find the internal chat service at
[https://chat.fribyte.no/](https://chat.fribyte.no/).

All you need to do is send an invitation to the new member. There are 2 ways to
invite someone: send an email or send an invitation link. The procedure is the
same.

1. Log in to the chat service with an admin user.
2. Click the name of the channel (at the very top of the channel list on the
   left), and choose `invite people` from the menu.
3. From there you can either send an email or generate a link.
4. Remember to add the new member to all channels.

## HeadScale (TailScale) VPN

[HeadScale](https://headscale.net) is an open-source version of TailScale. It
allows us to SSH directly to servers in Andeby without having to go through
Bestemor.

1. The new member must download the Tailscale client.
   [The method depends on the system](https://headscale.net/docs/installation/).

### MacOS HeadScale

https://headscale.net/apple-client

1. Download the official Windows client from the Tailscale website.
2. Run the installer.
3. Open the Tailscale app.
4. Do not sign in with your Tailscale account.
5. Open Terminal and run the following command:
   `/Applications/Tailscale.app/Contents/MacOS/Tailscale login --login-server=https://headscale.fribyte.no`
6. An existing friByte member needs to do the following:
   - Connect to the HeadScale server.
   - SSH to the HeadScale virtual machine with `ssh fribyte@headscale`.
   - Run `sudo ./join.sh <USERNAME> <mkey:LONGKEY>`.
     - Replace `<USERNAME>` with the new member's chosen username.
     - Replace `<mkey:LONGKEY>` with the new member's public key from the
       browser window the new member got on their PC.
7. The new member should now get a `Success` message in the Powershell window.
8. The new member can now connect to the VPN by running
   `/Applications/Tailscale.app/Contents/MacOS/Tailscale up -login-server https://headscale.fribyte.no --accept-routes`
   in Powershell.

### Windows HeadScale

https://headscale.net/windows-client/

1. Download the official Windows client from the Tailscale website.
2. Run the installer.
3. Open the Tailscale app.
4. Do not sign in with your Tailscale account.
5. Open Powershell and run the following command:
   `tailscale login --login-server=https://headscale.fribyte.no`
6. An existing friByte member needs to do the following:
   a. Connect to the HeadScale server.
   b. SSH to the HeadScale virtual machine with `ssh fribyte@headscale`.
   c. Run `sudo ./join.sh <USERNAME> <mkey:LONGKEY>`.
   - Replace `<USERNAME>` with the new member's chosen username.
   - Replace `<mkey:LONGKEY>` with the new member's public key from the browser
     window the new member got on their PC.
7. The new member should now get a `Success` message in the Powershell window.
8. The new member can now connect to the VPN by running
   `tailscale up -login-server https://headscale.fribyte.no --accept-routes` in
   Powershell.

### Linux HeadScale

https://headscale.net/running-headscale-linux

- Bro, you run Linux, you know what to do.

## Proxmox

Proxmox is the software we use for virtualization. To use the browser-based user
interface, you need your own user account.

1. Connect to the HeadScale VPN.
2. Log in with an admin user
   [on this site](https://proxmox:8006/#v1:0:18:4:::::8::14).
3. Click `Datacenter`.
4. Under `Permissions`, click `Users`.
5. Click `Add` and fill in the member's information.
6. Profit.
7. The new member can now log in at
   [https://proxmox:8006](https://proxmox:8006), as long as they are connected
   to the HeadScale VPN.

## GitHub

At the moment, all our repositories are on GitHub.
[You can find them here](https://github.com/fribyte-code). For the new member
to be able to contribute to the repositories, their GitHub account must be
added to the organization.

1. Go to [https://github.com/fribyte-code](https://github.com/fribyte-code).
2. Click `People`, then `Invite member`.
3. Search for the new member's user.
4. Profit.

Here are some things the new member should do:

1. Add themselves to the fribyte.no member list (`/content/kontakt/members.json`)
2. Add themselves to the admin repo member list (`/medlemmer/medlemsliste.csv`)

## Card access to the server room

To give the new member card access to the server room, the card office must be
contacted. They need to be sent a list of card numbers, as well as the room
number the card should have access to.

As a good Git exercise, the new member can do this themselves. Here is how:

1. Clone, or pull, the latest version of our GitHub repository called `admin`.
2. Open the file `medlemmer/legitimasjon/kortnummer` in your preferred text
   editor.
3. Enter your card number, save, and close the file.
4. Stage the changes and push them to `master`.
5. The chairperson must then send an email to the card office with a list of the
   card numbers; see the leader handbook in the admin repo. It would be nice if
   you notify the chairperson about this.

## Signal

If Mattermost is down, Signal is the backup communication platform. The new
member must download Signal and be added to the group chat.

## Andeby (Optional; not needed if you have HeadScale)

Andeby is the name of our server ecosystem. You get access to Andeby through the
[SSH protocol](https://www.ssh.com/academy/ssh).

You gain access to the ecosystem by SSHing to a specific domain.

1. If the new member does not have an SSH key, one must be generated.

`ssh-keygen -t ed25519 -C "your_email@example.com"` to generate an SSH key. Get
more information:
[GitHub docs, how to generate a new ssh key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).

2. Get the `.pub` key from the new member, print the public key with
   `cat .ssh/*.pub`, and send it to an active member in the friByte chat.

3. The next step must be done by an active friByte member who has access to
   Andeby.

4. Connect to Bestemor via `ssh root@andeby.fribyte.no`. This is the gateway to
   the rest of our beloved machines.

5. Open `authorized_keys` in your favorite code editor, for example nano:
   `nano ~/.ssh/authorized_keys`.

6. Write the name of the new member and paste in the SSH key.

## DEPRECATED Wireguard

[Wireguard](https://www.wireguard.com/) is a VPN service we use to connect to
some operations services. You must configure the connection between the server
and the new member's device. You can find a template for the client's config
file on `load-balancer-1`, under `/etc/wireguard`.

1. The new member must download Wireguard.
   [The method depends on the system](https://www.wireguard.com/install/).
2. The new member must generate new Wireguard keys.

```
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
```

3. From Bestemor: `ssh root@158.37.6.48` (that is, go to `load-balancer-1`)
4. `sudo vim /etc/wireguard/wg0.conf`
5. Run the script in the home directory on `load-balancer-1`:
   `./propagate_haproxy_wg.sh`
6. Create a new `[Peer]` in the form:

```
[Peer] # *member name*, *device type*
PublicKey = *the new member's public key*
AllowedIPs = 10.100.10.xx/32
```

`xx` must be replaced with a number that is not already occupied by another
`[Peer]`.

7. The new member must save a local Wireguard config file. It can be found in
   `/etc/wireguard` on Gjertrud. NB: Remember that the server's public key must
   be in the local config file, together with the private key that the new
   member generated.
   - Example of a local Wireguard file:

```toml
[Interface]
PrivateKey = xxxxx
ListenPort = 51871
Address = 10.100.10.xx/32

[Peer]
PublicKey = pEz/Wfumr/d9dEMjti3+Jf1KOtHVpd+zt8fW0pU1tmI=
AllowedIPs = 10.100.10.0/24
Endpoint = 158.37.6.28:51871
```

For the new member to connect to the Wireguard server:

8. On `load-balancer-1`: `wg-quick down wg0 && wg-quick up wg0`
9. Locally: `wg-quick up *config-file-name*`
10. To see that Wireguard works, visit https://proxmox.fribyte.no:8006

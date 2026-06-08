+++
title = "Beszel"
description = "Add a new Beszel system"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/instrukser/beszel.md"
+++

## Add a new Beszel system

This can be added with docker or with a binary (binary recommended).

### With Docker

- Log in to [beszel.fribyte.no](https://beszel.fribyte.no)
- Click "+ Add System"
- Select docker, enter a suitable name in the Name field. Fill in the IP
  address in the "Host / IP" field. It can, among other ways, be found by
  SSHing into the server and typing `curl config.me` in the terminal.
- Copy the beszel-agent contents (`Copy docker compose`). [To add
  systemd](https://www.beszel.dev/guide/systemd), add the following:

  - Under volumes, add:
  
  ```md
  /var/run/dbus/system_bus_socket:/var/run/dbus/system_bus_socket:ro
  ```

  - At the bottom, add:

```md
security_opt:
  - apparmor:unconfined
```

- SSH into the server you want to add, and paste what you copied from Beszel
  in a suitable place in `docker-compose.yml`.
- In the terminal on the server, to update docker: `docker compose pull "beszel-agent"` and `docker compose up -d "Image/beszel-agent"`

### With Binary

- Log in to [beszel.fribyte.no](https://beszel.fribyte.no)
- Click "+ Add System"
- Select binary, enter a suitable name in the Name field. Fill in the IP
  address in the "Host / IP" field. It can, among other ways, be found by
  SSHing into the server and typing `curl config.me` in the terminal
- Click "Copy Linux command"
- Paste it into the terminal on the server to add it

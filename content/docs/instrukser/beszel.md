+++
title = "Beszel"
description = "Legge til nytt Beszel-system"
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

## Legge til nytt Beszel system

Kan legges til med docker, eller med binary (binary anbefales)

### Med Docker

- Logg inn på [beszel.fribyte.no](https://beszel.fribyte.no)
- Trykk på "+ Add System"
- Velg docker, skriv inn passende navn på feltet Name. Fyll inn IP addressen på feltet "Host / IP". Kan bl.a finnes ved å ssh'e på serveren og skrive `curl config.me` i terminalen.
- Kopier innholdet til beszel-agent (`Copy docker compose`). [For å legge til systemd](https://www.beszel.dev/guide/systemd), legg til følgende:

  - Under volumes, legg til:
  
  ```md
  /var/run/dbus/system_bus_socket:/var/run/dbus/system_bus_socket:ro
  ```

  - Nederst, legg til:

```md
security_opt:
  - apparmor:unconfined
```

- SSH inn på serveren du vil legge til, lim inn det du har kopiert fra Beszel en passende plass i `docker-compose.yml`.
- I terminal på serveren for å oppdatere docker: `docker compose pull "beszel-agent"` og `docker compose up -d "Image/beszel-agent"`

### Med Binary

- Logg inn på [beszel.fribyte.no](https://beszel.fribyte.no)
- Trykk på "+ Add System"
- Velg binary, skriv inn passende navn på feltet Name. Fyll inn IP addressen på feltet "Host / IP". Kan bl.a finnes ved å ssh'e på serveren og skrive `curl config.me` i terminalen
- Trykk på "Copy Linux command"
- lim inn i terminal på serveren for å legge til

+++
title = "Github runner"
description = "Guffen self-hosted github actions runner"
date = 2024-05-07
updated = 2024-05-07
template = "docs/page.html"
sort_by = "weight"
weight = 3
+++

Guffen er en VM på gjertrud satt opp til å kjøre github actions tasks.

## Oppsett
Runneren er satt opp på [metoden oppgitt av GitHub](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners#adding-a-self-hosted-runner-to-a-repository)

Startskriptet for programmet ligger i `/home/fribyte/actions-runner/run.sh`

### Systemd service
For at programmet skal kjøre i bakgrunnen og starte av seg selv er det opprettet en systemd service som ligger i `/home/fribyte/.config/systemd/user/actions-runner.service`:

```
[Unit]
Description=Github actions runner
After=network.target

[Service]
ExecStart=/bin/bash /home/fribyte/actions-runner/run.sh
Type=notify-reload
Restart=always

[Install]
WantedBy=default.target
```

Følgende kommando kan brukes for *enable/start/stop* av servicen:
```bash
systemctl --user enable actions-runner.service
```

## Bruk
For at runneren skal brukes for å kjøre actions må man sette *runs-on* til self-hosted i actions filen.

 f.eks:
```yaml
jobs:
  push_to_registry:
    name: Push Docker image to Docker Hub
    runs-on: self-hosted
```

Hvis actions scriptet inneholder et docker build step kreves ogsa følgende som et step før byggingen i filen.
```yaml
        - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
```
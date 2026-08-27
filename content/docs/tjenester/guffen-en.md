+++
title = "GitHub runner"
description = "Guffen self-hosted GitHub Actions runner"
template = "docs/page.html"
sort_by = "weight"
weight = 10003

[extra]
lang = "en"
translation = "docs/tjenester/guffen.md"
+++

Guffen is a VM on Gjertrud set up to run GitHub Actions tasks.

## Setup

The runner is set up using
[the method described by GitHub](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners#adding-a-self-hosted-runner-to-a-repository).

The start script for the program is located at
`/home/fribyte/actions-runner/run.sh`.

### Systemd service

To make the program run in the background and start automatically, a systemd
service has been created at
`/home/fribyte/.config/systemd/user/actions-runner.service`:

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

The following command can be used to _enable/start/stop_ the service:

```bash
systemctl --user enable actions-runner.service
```

## Usage

To make the runner execute Actions jobs, you need to set `runs-on` to
`self-hosted` in the workflow file.

For example:

```yaml
jobs:
  push_to_registry:
    name: Push Docker image to Docker Hub
    runs-on: self-hosted
```

If the Actions script contains a Docker build step, the following step is also
required before the build in the workflow file.

```yaml
        - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
```

## Security

Using a self-hosted runner also comes with certain rules that should be
maintained to avoid security issues. This is because the runner executes
arbitrary code that could in theory escape the sandbox environment. To avoid
such situations, it is recommended to only allow Actions scripts in public
repositories to run after an approved merge to the `main` branch.

For future reference, see the
[GitHub runner security documentation](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners#self-hosted-runner-security).

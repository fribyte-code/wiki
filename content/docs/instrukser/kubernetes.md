+++
title = "Kubernetes"
description = "Kubernetes"
date = 2023-12-05T15:51:00+00:00
updated = 2023-12-05T15:51:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

# Kubernetes

I desember 2023 startet vi oppsett av Kubernetes klusteret vårt.

Målet er å gjøre tjenestene enda mer robuste og enklere å vedlikeholde. Vi
ønsker og å bruke Kubernetes for å lære mer om teknologien og da det er veldig
relevant for arbeidsliv.

- https://docs.k3s.io/datastore/ha-embedded
- [ ] En/to HaProxy node som lastbalanserer mellom server nodene
- [x] Tre server noder (Kontrollerer klustert)
  - Petter.fribyte.no
  - Raptus.fribyte.no
  - Hutre.fribyte.no
- [x] En agent node per fysisk server (der kubernetes podene (tjenester) kjører)
  - lille-hjelper-1.fribyte.no
  - lille-hjelper-1.fribyte.no
  - lille-hjelper-1.fribyte.no
- [ ] Rancher GUI for å administrere Kubernetes klusteret

## Oppsett av kluster

Argumenter til kommandoene under:

- `K3S_TOKEN` en slags api token som alle noder må vite
- `INSTALL_K3S_VERSION` versjon av k3s som skal installeres
- `--cluster-cidr` IPv6 subnett for pods (Viktig at nodene er innenfor samme
  subnett). Bruk /56 subnett /64 virker ikke.
- `--service-cidr` IPv6 subnett for tjenester (services) (må være laver enn
  128bits). Bruk /112 subnett

1. Starte cluster:

```sh
curl -sfL https://get.k3s.io | K3S_TOKEN=<TOKEN> INSTALL_K3S_VERSION=v1.26.10+k3s2 sh -s --server --cluster-cidr=2001:700:201:1:5001::/56 --service-cidr=2001:700:201:1:5001:3e3::/112 --cluster-init --disable=metrics-server
```

2. Joine some server noder:

```sh
curl -sfL https://get.k3s.io | K3S_TOKEN=<TOKEN> INSTALL_K3S_VERSION=v1.26.10+k3s2 sh -s server --server https://[2001:700:201:1:5001::2]:6443 --cluster-cidr=2001:700:201:1:5001::/56 --service-cidr=2001:700:201:1:5001:3e3::/112 --disable=metrics-server
```

- `--server` IP addresse til en av server nodene
- `-s` sier om man skal være agent eller server node

3. Joine som agent node

```sh
curl -sfL https://get.k3s.io | K3S_TOKEN=<TOKEN> INSTALL_K3S_VERSION=v1.26.10+k3s2 sh -s agent --server https://[2001:700:201:1:5001::2]:6443
```

- `--server` IP addresse til en av server nodene
- `-s` sier om man skal være agent eller server node

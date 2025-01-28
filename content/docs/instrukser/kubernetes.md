+++
title = "Kubernetes"
description = "Kubernetes"
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

# Kubernetes

I november 2024 startet vi oppsett av (det nye) Kubernetes klusteret vårt.

Målet er å gjøre tjenestene enda mer robuste og enklere å vedlikeholde. Vi
ønsker og å bruke Kubernetes for å lære mer om teknologien og da det er veldig
relevant for arbeidsliv.

- https://docs.k3s.io/datastore/ha-embedded
- [x] En/to HaProxy node som lastbalanserer mellom server nodene (lb-1 & lb-2 på
      letti & netti proxmox clusteret)
- [x] Tre server noder (Kontrollerer klustert)
  - Petter.fribyte.no
  - Raptus.fribyte.no
  - Hutre.fribyte.no
- [x] En agent node per fysisk server (der kubernetes podene (tjenester) kjører)
  - lille-hjelper-1.fribyte.no
  - lille-hjelper-1.fribyte.no
  - lille-hjelper-1.fribyte.no
- [ ] Letsencrypt inni Kubernetes for å utstede sertifikater til tjenestene
- [ ] Simple static docker container eksempel

## Oppsett av kluster

[Installere k3sup](https://github.com/alexellis/k3sup?tab=readme-ov-file#download-k3sup-tldr)
på Pluto, som har tilgang til NAT nettverket.

Installer master og agent nodene med k3sup på Pluto:

[Basert på denne kommandoen](https://github.com/alexellis/k3sup?tab=readme-ov-file#create-a-multi-master-ha-setup-with-embedded-etcd)

```bash
export USER=fribyte
export K3S_VERSION="v1.31.2+k3s1"

# Tailscale IP of the ha proxy load balancer, needed to access kubectl over tailscale
export HA_PROXY_TAILSCALE_IP=100.64.0.43

# Server nodes
export RAPTUS_IP=10.0.0.70
export PETTER_IP=10.0.0.71
export HUTRE_IP=10.0.0.72

# Worker/agwnt nodes
export LILLE_HJELPER_1_IP=10.0.0.80
export LILLE_HJELPER_2_IP=10.0.0.81
export LILLE_HJELPER_3_IP=10.0.0.82

# Master noder
k3sup install --ip $RAPTUS_IP --user $USER --cluster --k3s-version $K3S_VERSION --k3s-extra-args "--tls-san $HA_PROXY_TAILSCALE_IP"
k3sup join --ip $PETTER_IP --user $USER --server-user $USER --server-ip $RAPTUS_IP --server --k3s-version $K3S_VERSION --k3s-extra-args "--tls-san $HA_PROXY_TAILSCALE_IP"
k3sup join --ip $HUTRE_IP --user $USER --server-user $USER --server-ip $RAPTUS_IP --server --k3s-version $K3S_VERSION --k3s-extra-args "--tls-san $HA_PROXY_TAILSCALE_IP"

# Agent noder
k3sup join --ip $LILLE_HJELPER_1_IP --user $USER --server-user $USER --server-ip $RAPTUS_IP  --k3s-version $K3S_VERSION
k3sup join --ip $LILLE_HJELPER_2_IP --user $USER --server-user $USER --server-ip $RAPTUS_IP --k3s-version $K3S_VERSION
k3sup join --ip $LILLE_HJELPER_3_IP --user $USER --server-user $USER --server-ip $RAPTUS_IP --k3s-version $K3S_VERSION


mv ./.kube/config ./.kube/config-$(date +%s) # Backup existing kubeconfig
mv kubeconfig ./.kube/config # WARNING - This will overwrite your existing kubeconfig
kubectl label node lille-hjelper-1 kubernetes.io/role=agent
kubectl label node lille-hjelper-2 kubernetes.io/role=agent
kubectl label node lille-hjelper-3 kubernetes.io/role=agent
```

You can now access the Kubernetes cluster from your local machine by copying the
`~/.kube/config` file from Pluto to your local machine to the home directory.

- And replace `server: https://10.0.0.70:6443` with
  `server: https://100.64.0.43:6443`
- Confirm it is working using `kubectl get nodes`

+++
title = "Kubernetes"
description = "Kubernetes"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/instrukser/kubernetes.md"
+++

# Kubernetes

In November 2024, we started setting up our (new) Kubernetes cluster.

The goal is to make the services even more robust and easier to maintain. We
also want to use Kubernetes to learn more about the technology, since it is
highly relevant in working life.

- https://docs.k3s.io/datastore/ha-embedded
- [x] One/two HAProxy nodes load balancing between the server nodes (lb-1 & lb-2 on
      the letti & netti Proxmox cluster)
- [x] Three server nodes (controlling the cluster)
  - Petter.fribyte.no
  - Raptus.fribyte.no
  - Hutre.fribyte.no
- [x] One agent node per physical server (where the Kubernetes pods/services run)
  - lille-hjelper-1.fribyte.no
  - lille-hjelper-1.fribyte.no
  - lille-hjelper-1.fribyte.no
- [ ] Let's Encrypt inside Kubernetes to issue certificates for the services
- [ ] Simple static Docker container example

## Cluster setup

[Install k3sup](https://github.com/alexellis/k3sup?tab=readme-ov-file#download-k3sup-tldr)
on Pluto, which has access to the NAT network.

Install the master and agent nodes with k3sup on Pluto:

[Based on this command](https://github.com/alexellis/k3sup?tab=readme-ov-file#create-a-multi-master-ha-setup-with-embedded-etcd)

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

You can now access the Kubernetes cluster from your local machine by copying
the `~/.kube/config` file from Pluto to your local machine's home directory.

- Replace `server: https://10.0.0.70:6443` with
  `server: https://100.64.0.43:6443`
- Confirm that it works using `kubectl get nodes`

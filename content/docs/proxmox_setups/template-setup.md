+++
title = "Proxmox VM template setup"
description = "Guide for oppsett av nye VM templates i Proxmox VE"
template = "docs/page.html"
sort_by = "weight"
weight = 1
draft = false
+++

# Proxmox vm template setup

Denne prosessen er basert på følgende guider med modifiserte kommandoer for vårt oppsett:
- https://pve.proxmox.com/wiki/Cloud-Init_Support#_preparing_cloud_init_templates
- https://thepaulo.medium.com/create-an-ubuntu-24-04-template-with-cloud-init-on-proxmox-f092080cecfb

# Valg av cloud image

Først må man skaffe et oppdatert cloud image, vi bruker [ubuntu server](https://cloud-images.ubuntu.com/) per dags dato, men andre alternativer er også tilgjengelige.

Dette imaget må så lastes ned på Proxmox VE hosten med følgende kommando, hvor man erstatter med det nyeste imaget:

```bash
wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img
```

# Oppsett av VM

Så lager man en ny VM som skal brukes som template

VM id'en må være unik og må derfor endres fra 9000 i alle kommandoer

```bash
qm create 9000 --memory 2048 --net0 virtio,bridge=vmbr0 --scsihw virtio-scsi-pci
```
Man kan så importere imaget inn på VM'en
```bash
qm set 9000 --scsi0 basseng:0,import-from=/root/noble-server-cloudimg-amd64.img
```

```bash
qm resize 9000 scsi0 20G
```

```bash
qm set 9000 --scsihw virtio-scsi-pci --scsi0 basseng:vm-9000-disk-0
```

```bash
qm set 9000 --boot c --bootdisk scsi0
```

```bash
qm set 9000 --ide2 basseng:cloudinit
```

# Oppsett av VM instillinger og cloud init

Vi ønsker å sette noen standard instillinger for nye VM'er

## Under Hardware:

processors -> total cores = 2

## Under Cloud-init

user -> fribyte

pass -> ***

dns -> use host settings

ssh public key -> importer Proxmox VE host nøkler

IP config -> 
- **ipv4:** 158.37.6.0/26 gateway: 158.37.6.33
- **ipv6:** DHCP

## Under Options

OS Type -> Linux 6.x

# Konverter vm til template

Høyreklikk VM'en i listen til venstre > convert to template.

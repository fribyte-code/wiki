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
- [https://pve.proxmox.com/wiki/Cloud-Init_Support#_preparing_cloud_init_templates](https://pve.proxmox.com/wiki/Cloud-Init_Support#_preparing_cloud_init_templates)
- [https://thepaulo.medium.com/create-an-ubuntu-24-04-template-with-cloud-init-on-proxmox-f092080cecfb](https://thepaulo.medium.com/create-an-ubuntu-24-04-template-with-cloud-init-on-proxmox-f092080cecfb)

# Valg av cloud image

Først må man skaffe et oppdatert cloud image, vi bruker [ubuntu server](https://cloud-images.ubuntu.com/) per dags dato, men andre alternativer er også tilgjengelige.

Dette imaget må så lastes ned på Proxmox VE hosten med følgende kommando, hvor man erstatter med det nyeste imaget:

```bash
wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img
```

# Oppsett av VM

VM id erstattes med en VM id som ikke allerede er tatt i proxmox. Dette er fordi den nye VM id'en må være unik.
```bash
VM_ID="9000"
```
Vi oppretter en ny VM
```bash
qm create $VM_ID --memory 2048 --net0 virtio,bridge=vmbr0 --scsihw virtio-scsi-pci
```
Vi importerer cloud imagen inn på VM'en
```bash
qm set $VM_ID --scsi0 basseng:0,import-from=/root/noble-server-cloudimg-amd64.img
```
Vi endrer størrelsen på harddriven
```bash
qm resize $VM_ID scsi0 20G
```
Vi konfigurerer harddrive typen
```bash
qm set $VM_ID --scsihw virtio-scsi-pci --scsi0 basseng:vm-$VM_ID-disk-0
```
Vi setter harddriven til bootdisk
```bash
qm set $VM_ID --boot c --bootdisk scsi0
```
Vi legger til cloudinit driven
```bash
qm set $VM_ID --ide2 basseng:cloudinit
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

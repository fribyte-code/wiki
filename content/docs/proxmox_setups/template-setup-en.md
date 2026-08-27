+++
title = "Proxmox VM template setup"
description = "Guide for setting up new VM templates in Proxmox VE"
template = "docs/page.html"
sort_by = "weight"
weight = 10001
draft = false

[extra]
lang = "en"
translation = "docs/proxmox_setups/template-setup.md"
+++

# Proxmox VM template setup

This process is based on the following guides, with modified commands for our
setup:
- [https://pve.proxmox.com/wiki/Cloud-Init_Support#_preparing_cloud_init_templates](https://pve.proxmox.com/wiki/Cloud-Init_Support#_preparing_cloud_init_templates)
- [https://thepaulo.medium.com/create-an-ubuntu-24-04-template-with-cloud-init-on-proxmox-f092080cecfb](https://thepaulo.medium.com/create-an-ubuntu-24-04-template-with-cloud-init-on-proxmox-f092080cecfb)

# Choosing a cloud image

First you need to obtain an updated cloud image. At the moment we use [Ubuntu Server](https://cloud-images.ubuntu.com/), but other alternatives are also available.

The image must then be downloaded on the Proxmox VE host with the following command, replacing it with the latest image:

```bash
wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img
```

# Setting up the VM

`VM_ID` is replaced with a VM ID that is not already taken in Proxmox. This is because the new VM ID must be unique.
```bash
VM_ID="9000"
```
We create a new VM
```bash
qm create $VM_ID --memory 2048 --net0 virtio,bridge=vmbr0 --scsihw virtio-scsi-pci
```
We import the cloud image into the VM
```bash
qm set $VM_ID --scsi0 basseng:0,import-from=/root/noble-server-cloudimg-amd64.img
```
We change the size of the hard disk
```bash
qm resize $VM_ID scsi0 20G
```
We configure the hard disk type
```bash
qm set $VM_ID --scsihw virtio-scsi-pci --scsi0 basseng:vm-$VM_ID-disk-0
```
We set the hard disk as the boot disk
```bash
qm set $VM_ID --boot c --bootdisk scsi0
```
We add the Cloud-Init drive
```bash
qm set $VM_ID --ide2 basseng:cloudinit
```

# Setting up VM options and Cloud-Init

We want to set some standard settings for new VMs.

## Under Hardware:

`processors -> total cores = 2`

## Under Cloud-Init

`user -> fribyte`

`pass -> ***`

`dns -> use host settings`

`ssh public key -> import Proxmox VE host keys`

`IP config ->`
- **ipv4:** `158.37.6.0/26` gateway: `158.37.6.33`
- **ipv6:** `DHCP`

## Under Options

`OS Type -> Linux 6.x`

# Convert VM to template

Right-click the VM in the list on the left > `convert to template`.

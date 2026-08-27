+++
title = "Access to the Filesystem Inside a VM Disk Image"
description = "Access to the filesystem inside a VM disk image"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/instrukser/aksess-til-filsystem-inni-vm-disk-image.md"
+++

The purpose of this guide is to make it possible for friByters to read data
belonging to VMs that, for some arbitrary reason, cannot be reached and read
from inside the machine. This is done on the host that contains the VM, which
means no data transfer that would complicate networking or storage is
necessary.

Disk images in Proxmox appear as somewhat special files, seemingly symlinks. I
have not been able to see what lies on the other side, and the file claims it
is 0 bytes large. You can discover the location of these files with the
following command:
`pvesm path [storage pool]:[vm-disk-id]`, for example:
`pvesm path basseng:vm-531-disk-0`.

Proxmox often knows about the different partitions on these disk images. They
then appear at the same location returned by the command, as `disknavn-partX`,
where X is the partition number. In practice, partition number 1 is the root
partition on VMs cloned from our base.

Using the `losetup` command, the image can then be mounted as a "loop device".
This lets you use standard tools (e.g. fdisk) to work with the disk as if it
were a physical disk installed in the server:

- To turn a disk image into a loop device: `losetup -f -P [path returnert]`
  e.g. `losetup -f -P /dev/zvol/basseng/vm-531-disk-0-part1`
- To list active loop devices: `losetup -l`
- To deactivate a loop device (does not affect the disk image):
  `losetup -d /dev/loopX`.

Finally, mount the filesystem: `mount --mkdir /dev/loopX /mnt/my-vm-disk`. You
should now (hopefully) find the root of the VM's filesystem at
`/mnt/my-vm-disk`.

## If you need to clone the disk

Use `dd if=[path returnert av proxmox-kommandoen] of=~/my-disk-image.raw`.
This can then be mounted as a disk for a VM. To resize it (increase by
10GiB): `qemu-img resize my-disk-image.raw +10G`.

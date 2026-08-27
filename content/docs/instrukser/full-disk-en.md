+++
title = "Fix 100% Full Disk"
description = "Fix 100% Full Disk"
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/instrukser/full-disk.md"
+++

## Fix 100% Full Disk

If the disk is completely full, the system will not function properly.

Use the command `df -h` to confirm that the disk is full.

In Proxmox, shut down the relevant VM and change the disk size by clicking
Hardware -> Hard Disk -> Resize Disk (button at the top)

Most likely the VM will not be able to resize the partition to the new size; to
fix this, you can boot into GParted.

### Resize in GParted

1. Hardware -> Add -> CD/DVD Drive
2. Select the gparted-live ISO file
3. Change the boot order so that GParted is at the top and enabled, under
   Options -> Boot order
4. Change the display to VirtIO-GPU under Hardware -> Display
5. Boot the VM, start GParted, and use the default settings.
6. Once GParted is up, right-click the partition you want to resize, and click
   resize/move.
7. Change new size to the maximum.
8. Click apply, then shut down.
9. Change the boot order back, optionally remove the CD/DVD, and set the Display
   mode back to serial terminal 0.

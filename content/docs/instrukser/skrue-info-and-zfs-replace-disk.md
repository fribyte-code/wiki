+++
title = "Skrue Info + Replace disk ZFS"
description = "Info about Skrue and how to replace a disk in ZFS"
date = 2022-09-22T08:00:00+00:00
updated = 2022-09-22
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

## ZFS Skrue

1. Find the name of the device in the zpool (they will probably have a different
   name from what its device name is)
   - `glabel status` and `gpart list | grep 'Geom name\|label'` to view name
     corresponding to the device.
   - The dalar names also correspond to their respective row and column.
     Example: Dalar 42, four down, two over from top left.
2. Offline the disk in zfs
   - `zpool offline NAMEOFPOOL NAMEOFDISK` ex `zpool offline dalar gpt/dalar22`
3. zpool status check if drive is offline, if it is. Remove it from the server.
4. Insert the new drive into the server
5. Create partition
   - `gpart create -s GPT DEVICENAME` ex `gpart create -s GPT da3` (device name
     is same as found in step 1)
6. `gpart add -t freebsd-zfs -l dalar42 -a 4096 da7`
7. `gpart list` to find the new device name ex: `da2`
8. `zpool replace NAMEOFPOOL OLD_DEVICE_NAME /dev/gpt/NEW_DEVICE_LABEL` ex
   `zpool replace dalar 10304366615656783234 /dev/gpt/dalar22`
   - Find the old_device_name by looking for the offline device in
     `zpool status`

### Skrue harddrive layout

|                   |             |       |     |
| ----------------- | ----------- | ----- | --- |
| boot?             | swap/cache? | cache | -   |
| 2023-01-31        | 2023-02-02  | -     | -   |
| 2023-02-07        | old         | -     | -   |
| 2023-01 (5400rpm) | 2022?       | -     | -   |

## Useful commands:

- `zpool status` : View status of disks

- `gpart list | grep 'Geom name\|label'` -> Relate zpool name to physical drive

## Skrue ZFS snapshots

Skrue is configured to take snapshots regularly, this can lead to eating up disk
space without us knowing where it went. To check how much space is used by
snapshots, run the following command:

```sh
zfs list -t filesystem -o space
zfs list -t snapshot
```

To delete snapshots, run the following command:

```sh
zfs destroy dalar@202309040102_znap_monthly
```

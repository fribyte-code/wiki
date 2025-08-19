+++
title = "Ny VM Tailscale"
description = "Koble ny VM til tailscale nettet"
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

## How to connect a new VM to the tailscale network so we can ssh to it

1. Install tailscale following their official guide
   - [https://tailscale.com/download](https://tailscale.com/download)
1. Then connect to the tailscale mesh VPN by running this command

   ```
   sudo tailscale up --login-server https://headscale.fribyte.no --ssh  --accept-dns=false
   ```

1. Then you want to copy the key from the output, it should look something like
   follows

   ```
   mkey:27d04d3c1c5e9........
   ```

1. Then while keeping that terminal active, ssh into the headscale VM

   ```
   ssh fribyte@headscale
   ```

1. Then run this command where NAME=name of VM and the `mkey:cccfdsdsfds.....`

   ```
   sudo ~/join.sh <NAME> <Machine Key>
   ```

1. Add the vmuser/vm-name to the `dest` key inside the `ssh` acl in
   `/etc/headscale/headscale_acl` on the headscale server

1. Check that the VM is accessible over tailscale by running

   ```
   ssh fribyte@<NAME>
   ```

## Empty output

If for some reason you get an empty output when running `tailscale up` you can
get it from the journal log of the headscale VM. This can be seen with the
command

```
sudo journalctl -xeu headscale.service
```

It should be within one of the last log entries

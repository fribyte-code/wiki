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

1. Check that the VM is accessible over tailscale by running

   ```
   ssh fribyte@<NAME>
   ```

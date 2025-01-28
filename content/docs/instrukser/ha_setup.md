+++
title = "High availability for VM"
description = ""
template = "docs/page.html"
sort_by = "weight"
weight = 3
+++

## Prerequisites

First, in order to set up HA, you need to have the VM hosted on one of the
following servers:

- Fergus
- Skaftetrynet
- Pluto

It is also extremely important to have the VM drive hosted on the pool "basseng"

Other than that, the setup for creating a vm is the same as can be found
[here](../ny-vm)

## How to enable HA for a vm

1. Set up the VM, as according to the guide [here](../ny-vm). **Ensure the drive
   of the VM is stored on the pool "basseng"**
2. Create a Replication job, this can be found under the `Replication` tab
   within the Proxmox UI after clicking the VM.
   - You need to create two of these, each pointing to one of the other servers
   - Set the target to one of the other servers
   - Schedule: \*/15 (replication every 15 minutes, this will only transfer
     changes so won't bog the network)
   - Enabled checked
   - Everything else default
3. Navigate to the `Datacenter` tab found in the top of the tree structure found
   on the left. Then click on `HA`
4. Then fill it in as follows:
   - Group: Main
   - VM: Select the VM you want to enable HA for
   - The rest default
5. After all these steps are complete, replication is now set up!

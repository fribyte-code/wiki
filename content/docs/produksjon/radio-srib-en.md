+++
title = "radio.srib.no"
description = "Overview of AzuraCast and IceCast2, which power SRIB's internet radio."
template = "docs/page.html"
sort_by = "weight"
weight = 10005
draft = false

[extra]
lang = "en"
translation = "docs/produksjon/radio-srib-no.md"
+++

### SRIB - AzuraCast & Icecast server

[AzuraCast](https://docs.azuracast.com/en/home) is a free open-source
program for running internet radio and more. It comes with a built-in
[Icecast server](https://icecast.org/) that is used to stream audio from one or
more sources to many clients. For example, thousands of website visitors.
This is used today to run [lytt.srib.no](lytt.srib.no) for SRIB.

### Hosting

The AzuraCast service is hosted on the VM `srib-radio` in the
[ProxMox cluster](/docs/maskiner/bolivar-skaftetrynet-pluto-cluster) as a
Docker Compose instance.

AzuraCast has an incredibly good installer that was used to set up the
system.
[AzuraCast Docker installation guide](https://docs.azuracast.com/en/administration/docker/multi-site-installation).
The files are located in the `/var/azuracast` directory.

To support hosting multiple Docker containers on the same VM, we used the steps
under `AzuraCast Multi-Site Feature`, which set up an nginx reverse proxy and
automatic SSL certification.

Remember that other Docker containers must be connected to the network: `azuracast_frontend`
so that the reverse proxy can see the containers.

### Features

AzuraCast has many fancy features, and these can be explored on the domain of
the `srib-radio` VM. One thing we may want to look at is mounting the SRIB NAS into
AzuraCast and setting up automatic playback of media files when no signal reaches
the Icecast server. This could potentially remove the need for the dedicated
machine in SRIB's server room.

AzuraCast also has a podcast feature that may be worth looking at.

There is also a
[streamers & DJs](https://docs.azuracast.com/en/user-guide/streamers-and-djs)
feature that is not working right now because of some port trouble. But it opens up the possibility for
multiple sources to stream audio in, and then you can switch and mix between
the streamers. Potentially something that could be used for outside broadcasts.

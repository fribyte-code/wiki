+++
title = "Aksess til filsystem inni VM Disk Image"
description = "Aksess til filsystem inni VM Disk Image"
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

Hensikten med denne veiledningen er å gjøre det mulig for friBytere å lese data
som hører til VM-er som av en vilkårlig grunn ikke kan nås og leses fra innsiden
av maskinen. Dette gjøres på hosten som inneholder VM-en, som fører til at ingen
dataoverføring som blir nettverks- eller lagringskompliserende er nødvendig.

Diskbilder i Proxmox vises som litt spesielle filer, tilsynelatende symlinker.
Jeg har ikke klart å se hva som ligger på andre siden, og filen hevder den er 0
bytes stor. Disse filene kan du oppdage lokasjon til gjennom følgende kommando:
`pvesm path [storage pool]:[vm-disk-id]`, for eksempel:
`pvesm path basseng:vm-531-disk-0`.

Ofte vet Proxmox om de forskjellige partisjonene på disse diskbildene. Da kommer
de opp på samme lokasjon returnert av kommandoen, som `disknavn-partX` der X er
partisjonsnummeret. Erfaringsmessig er det partisjon nummer 1 som er
root-partisjonen på VM-er klonet fra vår base.

Gjennom kommandoen `losetup` kan så dette bildet mountes som en "loop device".
Dette gjør at du kan bruke standardverktøy (feks. fdisk) for å jobbe med disken,
som om det var en fysisk disk plassert i serveren:

- For å gjøre et diskbilde til en loop device: `losetup -f -P [path returnert]`
  feks. `losetup -f -P /dev/zvol/basseng/vm-531-disk-0-part1`
- For å liste loop devices aktive: `losetup -l`
- For å deaktivere en loop device (påvirker ikke diskbildet):
  `losetup -d /dev/loopX`.

Til slutt, mount filsystemet: `mount --mkdir /dev/loopX /mnt/my-vm-disk`. Du
finner nå (forhåpentligvis) roten av filsystemet til VM-en på `/mnt/my-vm-disk`.

## Om du må klone disken

Bruk `dd if=[path returnert av proxmox-kommandoen] of=~/my-disk-image.raw`.
Dette kan så mountes om en disk til en VM. For å endre størrelse (øke med
10GiB): `qemu-img resize my-disk-image.raw +10G`.

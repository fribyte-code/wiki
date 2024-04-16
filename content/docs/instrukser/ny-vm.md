+++
title = "Ny VM"
description = "Ny VM"
date = 2022-01-18T08:00:00+00:00
updated = 2022-03-14
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

## Hvordan sette opp ny VM:

If this VM is meant to have High availability, please read the setup for that
before starting. The setup can be found [here](../ha-setup)

1. Koble til wireguard
1. Gå til proxmox [https://10.100.10.1:8006](https://10.100.10.1:8006)
   alternativt:
   [https://proxmox.fribyte.no:8006](https://proxmox.fribyte.no:8006)
1. Clone templaten `6000 (Clone me ubuntu)` (høyreklikk -> clone)
1. Definer navn, gjerne navn på kunde
1. Velg en ledig VM-ID
1. Sett target storage til `basseng`
1. Klikk "Clone"
1. Brukernavn: fribyte
1. Passord: det vanlige.
1. Definer ip under "cloud init"
   - Finn ledig ip i [/docs/nettverk/nettverk-oversikt](/docs/nettverk/nettverk-oversikt)
   - Alternativt må du gjøre litt pinging rundt om kring på 158.37.6.xx for å
     finne en ledig ip
1. Sjekk at SSH public key er definert, hvis dette ikke er tilfellet kan de kopieres fra en annen VM.
1. Start vm og vent på at den er klar.
1. Oppdater `~/.ssh/config` på Konrad til å inkludere en peker til nye vm for
   lettere ssh'ing. Alternativt kan man legge inn en DNS peker på kladden slik
   at man kan bruke `ssh fribyte@maskinnavn.fribyte.no`.
1. Du skal nå kunne `ssh` inn til den via konrad med `ssh {definert-navn}`

## Tildel mer lagringsplass

1. Velg din VM
1. Gå til **Hardware**
1. Marker **Hard Disk**
1. Klikk på **Resize disk** i toppmenyen
1. Spesifiser antall GB du ønsker å øke den med i sprettoppvinduet
1. Profit!

+++
title = "Ny VM"
description = "Ny VM"
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
1. Clone templaten `6000 (Clone me ubuntu)` på Skaftetrynet eller
   `1000 (VM 1000)` på Fergus (høyreklikk -> clone)
1. Definer navn, gjerne navn på kunde
1. Velg en ledig VM-ID
1. Sett target storage til `basseng`
1. Klikk "Clone"
1. Brukernavn: fribyte
1. Passord: det vanlige.
1. Definer ip under "cloud init" ved å trykke på `IP Config` og `Edit`
   - Finn ledig ip i
     [/docs/nettverk/nettverk-oversikt](/docs/nettverk/nettverk-oversikt)
   - Alternativt må du gjøre litt pinging rundt om kring på 158.37.6.xx for å
     finne en ledig ip
   - IPv4 instillingene skal være:
     - IPv4/CIDR: 158.37.6.xx/26
     - Gateway: 158.37.6.33
1. Sjekk at SSH public key er definert, hvis dette ikke er tilfellet kan de
   kopieres fra en annen VM.
1. Oppgrader gjerne disk størrelse: Hardware -> Hard Disk -> Resize disk -> Øk
   med noen få GB
1. Øk minne: Hardware -> Memory -> Edit -> 4096
1. Øk CPU kjerner: Hardware -> Processors -> Edit -> Cores: 4
1. Start vm og vent på at den er klar. Se progress ved å trykke på `Console` og
   se at den booter.

- Det er ofte problemer på dette steget, fordi template man kopierer fra ikke
  virker som den skal. Spør gjerne om hjelp her.

1. Legg gjerne inn en DNS peker på DNS slik at man kan bruke
   `ssh fribyte@maskinnavn.fribyte.no`.
1. Du skal nå kunne `ssh` inn til den via skaftetrynet med `ssh {ip-addresse}`
   eller `ssh {definert-navn}`

## Tildel mer lagringsplass

1. Velg din VM
1. Gå til **Hardware**
1. Marker **Hard Disk**
1. Klikk på **Resize disk** i toppmenyen
1. Spesifiser antall GB du ønsker å øke den med i sprettoppvinduet
1. Profit!

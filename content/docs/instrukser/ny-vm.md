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

If this VM is meant to have High availability, please read the setup for that before starting. The setup can be found [here](../ha-setup)

1. Koble til wireguard
1. Gå til proxmox [https://10.100.10.1:8006](https://10.100.10.1:8006)
1. Clone templaten `8000 (Clone me ubuntu)` (høyreklikk -> clone)
1. Definer navn
1. Velg en ledig VM-ID
1. Bytt "linked mode" til "full clone"
1. Klikk "Clone"
1. Definer ip under "cloud init"
   - Finn ledig ip på [inventar.fribyte.uib.no](https://inventar.fribyte.uib.no)
   - Alternativt må du gjøre litt pinging rundt om kring på 158.37.6.xx for å
     finne en ledig ip
1. Start vm og vent på at den er klar
1. Oppdater `~/.ssh/config` på Konrad til å inkludere en peker til nye vm for
   lettere ssh'ing
1. Du skal nå kunne `ssh` inn til den via konrad med `ssh {definert-navn}`

## Tildel mer lagringsplass

1. Velg din VM
1. Gå til **Hardware**
1. Marker **Hard Disk**
1. Klikk på **Resize disk** i toppmenyen
1. Spesifiser antall GB du ønsker å øke den med i sprettoppvinduet
1. Profit!

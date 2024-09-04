+++
title = "Konrad"
description = "Oversikt over Konrad"
template = "docs/page.html"
sort_by = "weight"
weight = 1
draft = false
+++

# Konrad

**NB!** Konrad har problemer med å lagre BIOS konfigurasjon. Det er VELDIG
VIKTIG at når Konrad rebootes så må man velge boot device. Gå inn i BIOS under
boot, velg boot option med "Linux" i navnet.

## Proxmox

Serveren kjører Proxmox VE 7.2, og ligger i samme cluster som gjertrud. Web
panelet kan nås via wireguard gjennom gjertrud. Ellers kan den nås via en ssh
tunnell:

    ssh -L 8006:158.37.6.28:8006 root@bestemor.ss.uib.no

og naviger til `localhost:8006` i nettleseren.

Planen er å kjøre replication med konrad og gjertrud. TODO

## Lagringsoppsett

Konrad har pr dags dato et Dell H200 SATA HBA kort ( flashet til LSI 9211-8i
IT-mode ), som er koblet opp mot SAS backplanen i serveren.

Det er 4 1TB SSD disker koblet til HBA kortet, og 4 1TB HDD disker koblet til
hovedkortet. Disse ligger i hvert sitt ZFS RAIDZ1 pool, i pools med navn rpool
og storpool respektivt. Proxmox installasjonen ligger på rpool.

## Dokumentasjon av hvordan konrad ble satt opp

Vi satte opp serveren med lagring som beskrevet under
[lagringsoppsett](#lagringsoppsett). Noen av diskene var satt opp i raid med
lvm, disse ble wipet ved å bruke `lvremove` og `mdadm`. Vi prøvde tidligere et
annet Alreca HBA kort, som ikke fungerte bra i denne serveren.

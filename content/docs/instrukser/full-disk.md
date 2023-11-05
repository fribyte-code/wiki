+++
title = "Fikse 100% full disk"
description = "Fikse 100% full disk"
date = 2022-03-27T08:00:00+00:00
updated = 2022-03-27T08:00:00+00:00
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

## Fikse 100% full disk

Dersom disken er stappfull, vil systemet ikke fungere skikkelig.

Bruk kommandoen `df -h` for å bekrefte at disken er full.

I Proxmox, skrur du av VMen det gjelder, og endrer diskstørrelsen ved å trykke
på Hardware -> Hard Disk -> Resize Disk (knapp øverst)

Sannsynligvis klarer ikke VMen å resize partisjonen til den nye størrelsen, for
å fikse dette kan du boote inn i GParted.

### Endre størrelse i GParted

1. Hardware -> Add -> CD/DVD Drive
2. Velg gparted-live iso filen
3. Endre boot order slik at gparted er øverst og enabled, under Options -> Boot
   order
4. Endre display til VirtIO-GPU under Hardware -> Display
5. Boot opp VMen, start GParted, bruk default settings.
6. Når GParted er oppe, høyreklikk på partisjonen du vil resize, og klikk
   resize/move.
7. Endre new size til maks.
8. Trykk apply, og ta en shutdown.
9. Endre boot order tilbake, eventuelt fjern CD/DVD, og sett tilbake Display
   mode til serial terminal 0.

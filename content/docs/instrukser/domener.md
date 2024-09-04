+++
title = "Domener"
description = "Hvordan sette opp domener"
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

## Hvordan sette opp nytt domene:

1. Ta turen til bestemor `ssh root@bestemor.s.fribyte.no`
2. `ssh fribyte@ns1.fribyte.no`
3. Oppdater passende domene ved å kjøre scriptet `./update_zone.sh fribyte.no`
   (bruk noe annet enn fribyte.no for å redigere domener som ikke er
   .fribyte.no)
4. Legg til passende entry
5. Oppdater serial `{år}{måned}{dato}{hh}` -> `2022020618`

- Viktig at serial økes!
  - Men serial kan ikke være større enn 32 bit

6. `rndc reload fribyte.no` -> Utførers automatisk av `update_zone.sh`
7. `rndc reload` -> Utførers automatisk av `update_zone.sh`
8. Kan i teorien ta opp til 2 timer før den virker, men tar gjerne 10-20
   minutter
9. Vi bruker Cloudflare sin dns `1.1.1.1` så det kan hjelpe å besøke
   https://1.1.1.1/purge-cache/ og så skrive inn ditt nye domenenavn og trykk
   "Purge Cache". Da vil Cloudflare oppdatere det domenet for alle i hele verden
   som bruker Cloudflare.

For mer detaljert informasjon, se [/docs/nettverk/dns](/docs/nettverk/dns)

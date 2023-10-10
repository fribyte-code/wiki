+++
title = "Uptime Kuma"
description = "Hvordan legge til en ny tjeneste i uptime kuma"
date = 2023-03-30
updated = 2022-04-18
template = "docs/page.html"
sort_by = "weight"
weight = 3
+++

#friByte bruker [Uptime Kuma](https://github.com/louislam/uptime-kuma) for holde oversikt over oppetiden til tjenestene vi drifter. Den er å finne på [uptime.fribyte.no](https://uptime.fribyte.no). Du logger inn på admin-panelet ved å gå til [uptime.fribyte.no/dashboard](https://uptime.fribyte.no/dashboard).

Denne tjenesten er koblet opp mot mattermost gjennom "Mattermost notification"-funksjonen til uptime kuma. 

### Hvordan legge til en ny tjeneste

1. Logg inn
2. Trykk på "legg til ny overvåkning"
3. Fyll inn følgende:

- Vennlig navn: Hva tjenesten skal hete i listen
- Forsøk: Hvor mange ganger uptime skal prøve på nytt før den sier ifra om en feil
- Certificate Expiry Notification: Sier ifra om letsencrypt har gått ut. 
- Oppetidroboten: om denne tjenesten skal sende varsel til mattermost

_Det finnes andre innstillinger også, men dette er minimum du trenger_

### Hvordan legge til tjeneste på statussiden

1. Logg inn
2. Gå til [statussiden](https://uptime.fribyte.no).
3. Trykk på "rediger statusside"
4. I feltet "Legg til overvåkning", skriv inn navnet på tjenesten
5. Flytt tjenesten til en hensiktsmessig gruppe

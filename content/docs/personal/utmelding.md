+++
title = "Melde ut medlem"
description = "Hvordan melde ut nytt medlem"
date = 2024-05-14T08:00:00+00:00
updated = 2024-05-14
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

Dette er en instruks for hvordan fjerne medlemmers tilgang til de relevante systemene våre. Dette må gjøres hver gang et medlem er satt til å være **inaktiv** eller slutter i friByte på ubestemt tid. Enkelte rutiner må også gjennomføres om medlemmet har **Guru**-status.

Avhengig av hva det gjeldende medlemmet ønsker, er det ulike rutiner som må gjennomføres.

## Migadu

Man kan logge inn til admin-panelet ved å gå til: [admin.migadu.com/login](https://admin.migadu.com/public/login). Man logger bare inn med sine egne bruker-kredentialer, men bruker må ha **administrator**-rettigheter.

### Fjerne admin-status

1. Logg inn som administrator på [admin.migadu.com/login](https://admin.migadu.com/public/login).

2. Trykk "My Organization" > "Administrators" > *Det gjeldende medlemmet* > "Delete Administrator..."

### Fjerne medlemmet fra listen over aktive epost-adresser

### Legge medlemmet til i listen over inaktive epost-adresser

### Om medlemmet er guru: Legge medlemmet til i listen over guru epost-adresser

### Om medlemmet ønsker videresending: Legge til en delegasjonsadresse

## Mattermost

Om vedkommende medlem skal fjernes fra Mattermost burde være en skjønnsmessig sak, siden det er en ganske enkel og god måte å nå ut til friByte igjen om det skulle gjelde noe. 

**En ting å tenke på, som burde inngå i den skjønnsmessige vurderingen, er at alle medlemmer som gis tilgang til Mattermost kan se hele historikken, filer, tidligere diskusjoner, samt alle medlemmer, inkludert de som ikke nødvendigvis er aktive.** 

Om de skal fjernes:

1. som medlem med administrator-status, gå til **System Console** > **Users** > *medlemmets bruker* > **Actions** > **Deactivate**

## SSH-tilgang

Gå til enhver fysiske/virtuelle maskin som har SSH-nøkkelen til medlemmet og åpne `authorized_keys`. Finn SSH-nøkkelen til medlemmet og kommentér den ut. 

## Wireguard

1. Gå til maskinen som kjører Wireguard-tjeneren.
2. Kommentér ut de offentlige nøklene til medlemmet i `wg0.config`.
3. Kommentér ut de tildelte IP-adressene til medlemmet.

Om medlemmet skal fjernes helt, kan de de offentlige nøklene og IP-adressene fjernes. 

## ProxMox

TBA

## Github

For øyeblikket er alle repoene våre på Github. [Du kan finne dem her](https://github.com/fribyte-code). For at det nye medlemmet skal kunne bidra til repoene, må deres github-konto legges til i organisasjonen.

1. Klikk deg inn på [https://github.com/fribyte-code](https://github.com/fribyte-code).
2. Trykk på **People**, deretter de tre prikkene og **Remove from organisation**.

Alternativt medlemmet gjøres til **Collaborator**. Da har de fremdeles tilgang til repoene de har bidratt til tidligere, men får ikke tilgang på noen av de nye private repoene.

## Korttilgang til serverrommet

Om de er UiB-student, må kortsenteret kontaktes for å kunne fjerne tilgangen deres. 

Om de ikke er student ved UiB, kan gjestekortet de fikk fra UiB konfiskeres(!)

## Signal

Å fjerne medlemmet fra Signal-gruppen blir en skjønnsmessig vurdering.

Om de skal fjernes:

1. Som administrator i gruppen, gå til **Gruppeinstillinger** > **Medlemmer** > *Medemmets bruker* > **Fjern fra gruppen**.



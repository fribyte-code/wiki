+++
title = "Nytt medlem"
description = "Hvordan innføre nytt medlem"
date = 2022-02-23T08:00:00+00:00
updated = 2022-08-31
template = "docs/page.html"
sort_by = "weight"
weight = 5
draft = false
+++

Dette er en instruks for hvordan gi nye medlemmer tilgang til de relevante
systemene våre.

## Migadu

Migadu er epost-tjenesten vi bruker internt.

Man kan logge inn til admin-panelet ved å gå til:
[admin.migadu.com/login](https://admin.migadu.com/public/login). Man logger bare
inn med sine egne bruker-kredentialer, men bruker må ha
**administrator**-rettigheter.

For å lage en ny epost-adresse for det nye medlemmet.

1. Logg inn med admin-bruker
2. Gå til "Email Domains" -> "friByte.no"
3. Under "All Addresses" eller "Mailboxes", trykk på "New Mailbox"
4. Fyll inn medlemmets info
5. Profitt

### Legge nytt medlem til i mailing listene:

1. Logg inn med admin-bruker
2. Gå til "Email Domains" -> "aktive@friByte.no"
3. Gå til Mailboxes
4. Velg fribyte@fribyte.no
5. Under delegation: Legg til ny epost addresse i tekstfeltet for Local
   Recipients

## Mattermost

Man finner den interne preiketjenesten om man går til
[https://chat.fribyte.no/](https://chat.fribyte.no/).

Alt som trengs er å sende en invitasjon til det nye medlemmet. Det er 2 måter
man kan invitere; Sende ebrev eller sende invitasjonslenke. Fremgangsmåten er
den samme.

1. Logg inn på preiketjenesten med an admin-bruker
2. Trykk på navnet på kanalen(helt øverst i kanal-listen til venstre), og velg
   "invite people" fra menyen.
3. Herfra kan man enten sende ebrev eller generere en lenke

## Andeby

Andeby er navnet på økosystemet vårt av maskintjenere. Man får tilgang til
Andeby gjennom [SSH-protokollen](https://www.ssh.com/academy/ssh).

Man får tilgang til økosystemet ved å SSH'e seg til et spesifikt domene.

`ssh root@andeby.fribyte.no` er gatewayen til resten av våre kjære maskiner.

1. Om det nye medlemmet ikke har en SSH-nøkkel, må det genereres.
   [Github docs, how to generate a new ssh key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).
2. Legg til den nye genererte nøkkelen til '~/.ssh/authorized_keys' i Bestemor.
   Dette er noe et aktivt medlem må gjøre.

## Wireguard

[Wireguard](https://www.wireguard.com/) er en VPN-tjeneste vi bruker for å koble
oss opp mot enkelte drift-tjenester. Man må konfigurere tilkoblingen mellom
maskintjeneren og det nye medlemmets enhet. Man kan finne en mal for klienten
sin konfigureringsfil på load-balancer-1, under `/etc/wireguard`.

1. Det nye medlemmet må laste ned Wireguard.
   [Metoden avhenger av systemet](https://www.wireguard.com/install/).
2. Det nye medlemmet må generere nye Wireguard-nøkler.

```
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
```

3. fra bestemor: `ssh root@158.37.6.49`
4. `vim /etc/wireguard/wg0.conf`
5. Lag en ny [Peer] i form av:

```
[Peer] # *navn på medlem*, *enhetstype*
PublicKey = *public-keyen til det nye medlemmet*
AllowedIPs = 10.100.10.xx/32
```

xx må byttes ut med et tall som ikke allerede er okkupert av en annen [Peer].

6. Det nye medlemmet må lagre en lokal Wireguard-konfigureringsfil. Den kan
   finnes i `/etc/wireguard` i gjertrud. NB: Husk at public-keyen til
   maskintjeneren skal være i lokal konfig-fil, samt private-keyen som det nye
   medlemmet genererte.
   - Eksempel på lokal wireguard fil:

```toml
[Interface]
PrivateKey = xxxxx
ListenPort = 51871
Address = 10.100.10.xx/32

[Peer]
PublicKey = pEz/Wfumr/d9dEMjti3+Jf1KOtHVpd+zt8fW0pU1tmI=
AllowedIPs = 10.100.10.0/24
Endpoint = 158.37.6.28:51871
```

For at det nye medlemmet skal koble seg på Wireguard-maskintjeneren:

7. På skaftetrynet: `wg-quick down wg0 && wg-quick up wg0`
8. lokalt: `wg-quick up *navn på konfig-fil*`
9. For å se at wireguard virker, besøk https://proxmox.fribyte.no:8006

## ProxMox

ProxMox er programvaren vi bruker for å virtualisering. For å kunne bruke
brukergrensesnittet i nettleseren, må man ha en egen bruker.

1. Logg inn med admin-bruker
   [på denne nettsiden](https://10.100.10.1:8006/#v1:0:18:4:::::8::14).
2. Trykk på "Datacenter"
3. Under "Permissions", trykk på "Users"
4. Trykk "Add" og fyll inn medlemmets info.
5. Profitt

## Github

For øyeblikket er alle repoene våre på Github.
[Du kan finne dem her](https://github.com/fribyte-code). For at det nye
medlemmet skal kunne bidra til repoene, må deres github-konto legges til i den
organisasjonen.

1. Klikk deg inn på
   [https://github.com/fribyte-code](https://github.com/fribyte-code).
2. Trykk på "People", deretter "Invite member".
3. Søk opp det nye medlemmets bruker.
4. Profitt.

## Korttilgang til serverrommet

For å gi det nye medlemmet korttilgang til serverrommet, så må kortsentralen
kontaktes. De må bli tilsendt en liste av kortnumre, samt romnummeret som kortet
skal ha tilgang til.

Som en god øvelse i git, kan det nye medlemmet gjøre det selv. Dette er hva en
gjør:

1. Klon, eller dra ned, siste versjon av github-repoet vårt som heter `admin`.
2. Åpne filen, `../legitimasjon/kortnummer` med ditt foretrukne
   tekstredigeringsprogram.
3. Skriv inn kortnummeret ditt, lagre og lukk filen.
4. Etabler forandringene og dytt dem til master.
5. Leder må dermed sende mail til kortsenteret med en liste av kortnummerene, se
   vitebok for leder i admin repoet.

## Signal

Ved tilfeller der Mattermost er nede så er Signal backup kommunikasjonsplatform.
Her må det nye medlemmet laste ned Signal og bli lagt til i gruppechatten.

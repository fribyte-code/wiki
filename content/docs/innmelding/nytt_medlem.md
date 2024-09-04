+++
title = "Nytt medlem"
description = "Hvordan innføre nytt medlem"
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
5. Trykk "Create Mailbox"
6. Profitt

### Legge nytt medlem til i mailing listene:

1. Logg inn med admin-bruker
2. Gå til Mailboxes
3. Gå til "List" -> "aktive@friByte.no"
4. Under Overview -> Delegation
5. Legg til ny epost addresse i tekstfeltet for Local
   Recipients
6. Lagre endringer
7. Enda mer profitt

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
4. Husk å legge til nytt medlem til alle kanaler.

## Andeby

Andeby er navnet på økosystemet vårt av maskintjenere. Man får tilgang til
Andeby gjennom [SSH-protokollen](https://www.ssh.com/academy/ssh).

Man får tilgang til økosystemet ved å SSH'e seg til et spesifikt domene.



1. Om det nye medlemmet ikke har en SSH-nøkkel, må det genereres.

`ssh-keygen -t ed25519 -C "your_email@example.com"` for å generere en SSH-nøkkel. Få mer info:
   [Github docs, how to generate a new ssh key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).

2. Hente .pub key fra det nye medlemmet, skriv ut public key med `cat .ssh/*.pub` og sende det til aktivt medlem i friByte chat. 

3. Neste steg må et aktivt medlem i friByte som har tilgang til Andeby gjøre

4. Koble til bestemor via `ssh root@andeby.fribyte.no`. Dette er gatewayen til resten av våre kjære maskiner. 

5. Åpne authorized_keys i ditt favoritt text editor, nano for eksempel: `nano ~/.ssh/authorized_keys`. 

6. Skriv navnet til det nye medlemmet og lim inn ssh-nøkkelen. 

7. Commence hostile takeover!


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

3. fra bestemor: `ssh root@158.37.6.48` (altså gå til load-balancer-1)
4. `sudo vim /etc/wireguard/wg0.conf`
5. Kjør scriptet i hjemmemappen på load-balancer-1: `./propagate_haproxy_wg.sh`
6. Lag en ny [Peer] i form av:

```
[Peer] # *navn på medlem*, *enhetstype*
PublicKey = *public-keyen til det nye medlemmet*
AllowedIPs = 10.100.10.xx/32
```

xx må byttes ut med et tall som ikke allerede er okkupert av en annen [Peer].

7. Det nye medlemmet må lagre en lokal Wireguard-konfigureringsfil. Den kan
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

8. På load-balancer-1: `wg-quick down wg0 && wg-quick up wg0`
9. lokalt: `wg-quick up *navn på konfig-fil*`
10. For å se at wireguard virker, besøk https://proxmox.fribyte.no:8006

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

Her er noen ting som bør gjøres av det nye medlemmet

1. Legge til seg selv på fribyte.no medlemsliste (/content/kontakt/members.json)
2. Legge til seg selv på admin-repoet medlemsliste (/medlemmer/medlemsliste.csv)

## Korttilgang til serverrommet

For å gi det nye medlemmet korttilgang til serverrommet, så må kortsentralen
kontaktes. De må bli tilsendt en liste av kortnumre, samt romnummeret som kortet
skal ha tilgang til.

Som en god øvelse i git, kan det nye medlemmet gjøre det selv. Dette er hva en
gjør:

1. Klon, eller dra ned, siste versjon av github-repoet vårt som heter `admin`.
2. Åpne filen `medlemmer/legitimasjon/kortnummer` med ditt foretrukne
   tekstredigeringsprogram.
3. Skriv inn kortnummeret ditt, lagre og lukk filen.
4. Etabler forandringene og dytt dem til master.
5. Leder må dermed sende mail til kortsenteret med en liste av kortnummerene, se
   vitebok for leder i admin repoet. Fint om du varsler leder om dette.

## Signal

Ved tilfeller der Mattermost er nede så er Signal backup kommunikasjonsplatform.
Her må det nye medlemmet laste ned Signal og bli lagt til i gruppechatten.

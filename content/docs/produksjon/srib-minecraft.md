+++
title = "SRiB Minecraft server"
description = "Alt du trenger å vite om minecraft-serveren til SRiB"
date = 2022-12-30T04:00:00+00:00
updated = 2024-02-10
template = "docs/page.html"
sort_by = "weight"
weight = 1
draft = false
+++

# SRiB Minecraft Server

Dette er et dokument som beskriver alt du trenger å vite om SRiB sin ;MineCraft server

## Installasjon

### Miljø

Operativsystemet er en fersk installasjon av Ubuntu 22.04 på en virtuell
maskin i ProxMox 8 Utenom det er det ingen spesielle miljø-konfigurasjoner.

### Prosessen

1. `sudo apt-get install lib32gcc-s1 lib32stdc++6 libsdl2-2.0-0 netcat screen -y`
2. `sudo add-apt-repository ppa:openjdk-r/ppa`
3. `sudo apt-get update`
4. `sudo apt-get install openjdk-17-jre-headless -y`
5. `sudo ufw allow 25565`
6. `sudo adduser mcserver`
7. `su - mcserver`
8. `wget -O linuxgsm.sh https://linuxgsm.sh`
9. `chmod +x linuxgsm.sh`
10. `bash linuxgsm.sh mcserver`
11. `~/mcserver install`
12. `sudo nano serverfiles/server.properties`
13. finn linjen: `server-ip:xxx.xxx.xxx.xxx` og skriv inn IP-adressen til
    serveren
14. lagre og lukk filen.
15. `~/mcserver start`
16. (ekstra) Du kan sjekke statusen til serveren ved å kjøre:
    `~/mcserver monitor`

Kilde:
[https://www.techrepublic.com/article/install-minecraft-server-ubuntu/](https://www.techrepublic.com/article/install-minecraft-server-ubuntu/)

Merk: spill-filene ligger, som følge av denne prosessen, under `/home/mcserver`
i den aktuelle VM-en. mcserver-brukeren har heller ikke sudo-rettigheter.

pakken `libsdl2-2.0-0` står skrevet i kilden som `libsdl2-2.0-0:1386`, men denne
pakken er spesifikt for Intel-maskiner.

Når du kjører `~/mcserver install`, blir avhengigheter sjekket. Siden
mcserver-brukeren ikke har sudo-rettigheter, kan ikke detet scriptet intallere
dem. Du må derfor installere dem manuelt. I så tifelle er det så enkelt som:
`sudo apt-get install <navn på pakker>`.

## Detaljer

VM-en kjører på Konrad, ligger lagret på local og er tildelt 4GB RAM og 4GB
lagring, med 2 cores og 2 sockets.

[Anbefalte spesifikasjoner for Linux-maskiner kan du finne her](<https://minecraft.fandom.com/wiki/Server/Requirements/Dedicated#Unix_(Linux,_BSD,_macOS)>)

[IP-adressen tildelt til serveren kan du finne her](@/docs/nettverk/nettverk-oversikt.md)

### Scripts

Her er systemtjenesten som er opprettet for å starte minecraft-serveren
automatisk etter en omstart.

```sh
[Unit]
Description=MineCraft-server for SRiB
After=network.target

[Service]
User=root
WorkingDirectory=/home/mcserver
ExecStart=/home/fribyte/startmcserver.sh "~/mcserver start" mcserver
Restart=always

[Install]
WantedBy=multi-user.target
```

NB: husk å kjøre `sudo systemctl enable mcsrib` etterfulgt av
`sudo systemctl daemon-reload`

Her er startserver.sh som starter tjenester

```sh
#!/usr/bin/expect -f
#Usage: script.sh cmd user pass

set cmd [lindex $argv 0];
set user [lindex $argv 1];

log_user 0
spawn su -c $cmd - $user
expect "Password: "
log_user 1
send "<passord>\r"
expect "$ "
```

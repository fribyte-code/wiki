+++
title = "SRiB Minecraft server"
description = "Everything you need to know about SRiB's Minecraft server"
template = "docs/page.html"
sort_by = "weight"
weight = 10001
draft = false

[extra]
lang = "en"
translation = "docs/produksjon/srib-minecraft.md"
+++

# SRiB Minecraft Server

This is a document that describes everything you need to know about SRiB's
Minecraft server.

## Installation

### Environment

The operating system is a fresh installation of Ubuntu 22.04 on a virtual
machine in ProxMox 8. Other than that, there are no special environment
configurations.

### Process

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
13. find the line: `server-ip:xxx.xxx.xxx.xxx` and enter the IP address of the
    server
14. save and close the file.
15. `~/mcserver start`
16. (optional) You can check the status of the server by running:
    `~/mcserver monitor`

Source:
[https://www.techrepublic.com/article/install-minecraft-server-ubuntu/](https://www.techrepublic.com/article/install-minecraft-server-ubuntu/)

Note: as a result of this process, the game files are located under
`/home/mcserver` on the relevant VM. The `mcserver` user also does not have
sudo privileges.

The package `libsdl2-2.0-0` is written in the source as `libsdl2-2.0-0:1386`,
but that package is specifically for Intel machines.

When you run `~/mcserver install`, dependencies are checked. Since the
`mcserver` user does not have sudo privileges, that script cannot install them.
You must therefore install them manually. In that case, it is as simple as:
`sudo apt-get install <navn på pakker>`.

## Details

The VM runs on Konrad, is stored on local, and is assigned 4GB RAM and 4GB
storage, with 2 cores and 2 sockets.

[You can find recommended specifications for Linux machines here](<https://minecraft.fandom.com/wiki/Server/Requirements/Dedicated#Unix_(Linux,_BSD,_macOS)>)

[You can find the IP address assigned to the server here](@/docs/nettverk/nettverk-oversikt.md)

### Scripts

Here is the system service that was created to start the Minecraft server
automatically after a reboot.

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

Note: remember to run `sudo systemctl enable mcsrib` followed by
`sudo systemctl daemon-reload`

Here is startserver.sh, which starts services

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

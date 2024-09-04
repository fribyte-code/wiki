+++
title = "Database-migrering"
description = "En forklaring av hvordan jeg migrerte alle databasene fra Donald (ubuntu 14.04) til en VM på Konrad (Ubuntu 22.02)."
template = "docs/page.html"
sort_by = "weight"
weight = 3
+++

friByte drifter SQL-databaser for klienter. Dette har vi forsåvidt alltid gjort,
men de nye databasene vi drifter ligger på VM-ene i ProxMox. Tidligere lå alle
databasene på en maskintjener som heter Donald. Siden mange av tidligere
klienter som brukte de databasene aktivt ikke lenger bruker dem, har databasen
stått til forfall.

Vi har derfor nylig (06.03.2023) flyttet alle databasene over til én VM i
ProxMox-clusteret, med det intuitive navnet "database". På den måten garanterer
vi for regelmessige sikkerhetskopier i tillegg til å modernisere databasen.

### Overføre databasene

1. Inne på Donald: `mysqldump --all-databases --replace --events > donald.sql`
2. Inne på Donald:
   `rsync -r -v --info=progress2 donald.sql root@158.37.6.28:/root`
3. Inne på Konrad:
   `rsync -r -v --info=progress2 /root/donald.sql fribyte@158.37.6.37:/home/fribyte`
4. Inne på VM-en: `sudo apt install pv`\*
5. Inne i mysql på VM-en: `SET GLOBAL innodb_strict_mode=OFF;`
6. Inne på VM-en: `pv donald.sql | sudo mysql -u root -p`
7. Inne på VM-en: `sudo service mysql stop`
8. Inne på VM-en: `sudo mysqld --upgrade=FORCE`
9. Inne på VM-en: `sudo service mysql start`

\*_"pv" står for "pipeviewer" og er bare for å kunne se fremgangen på
overføringen._

Når alt dette var gjennomført, ble innloggings-informasjonen og
bruker-informasjonen fra sql-dumpen overført inn i den ferske mysql-serveren.
Hvilket betyr at en kan logge inn i denne databasen med de samme brukerne som
den gamle databasen.

Men rettighetene fungerte ikke helt skikkelig, så jeg måtte også endre
root-passordet, for å få full admin-tilgang.

### Slik endret jeg root-passordet

1. Inne på VM-en: `sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf` og legg til
   `skip-grant-tables` under `[mysqld]`. Lagre og lukk.
2. Inne på VM-en: `sudo service mysl restart`
3. Inne på VM-en: `sudo mysql -u root`
4. Inne i mysql på VM-en: `FLUSH PRIVILEGES;`
5. Inne i mysql på VM-en:
   `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'lololol';`

Om du nå fjerner `skip-grant-tables` fra `/etc/mysql/mysql.conf.d/mysqld.cnf`,
og kjører `sudo service mysql restart`, kan du nå logge inn som `root` med
passord: `lololol`.

Etter dette, kunne jeg endre passord på brukerne som kom med `donald.sql`. Jeg
gjorde følgende:

1. Inne på VM-en: `sudo mysql -u -p` _skriv inn passord_
2. Inne i mysql på VM-en:
   `ALTER USER 'digas'@'%' IDENTIFIED WITH mysql_native_password BY 'lololol';`\*\*
3. Inne i mysql på VM-en: `FLUSH PRIVILEGES;`

\*\*_"digas" er database-brukeren til SRiB for DigaSystem-programmet deres. "%"
betyr hvilken som helst host_

### Full disk ila. overføring

Denne prosessen tok litt tid, og trengte en god del mer lagringsplass på VM-en
enn jeg forventet. Så overføringen kræsjet flere ganger på grunn av mangel på
plass.

SQL-dumpen, donald.sql, tok opp 5.1 GB med lagring, men når den var overført til
mysql-serveren, tok databasen opp 12 GB. Så, til sammen 17.1 GB.

Så dette er hvordan jeg håndterte full disk.

1. Inne på VM-en:
   `sudo apt-get purge mysql-server mysql-client mysql-common mysql-server-core-* mysql-client-core-* && sudo rm -rf /etc/mysql /var/lib/mysql`
2. Inne på VM-en: `sudo apt install mysql-server -y`
3. Inne i mysql: `SET GLOBAL innodb_strict_mode=OFF;`
4. Tildel VM-en mer lagringsplass. Dette kan gjøres i web GUI-en til ProxMox.
5. Inne på VM-en: `pv donald.sql | sudo mysql -u root -p`

### Åpne for eksterne tilkoblinger

For at eksterne maskiner skal kunne koble seg til, må vi skrive dette inn i
konfigureringen.

1. Inne på VM-en: `sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf`
2. Endre:

```bash
bind-address            = 127.0.0.1
mysqlx-bind-address     = 127.0.0.1
```

til å være:

```bash
bind-address            = 0.0.0.0
mysqlx-bind-address     = 0.0.0.0
```

Da åpner du bare for alle IPv4-addresser. Alternativt kan du også endre det til:

```bash
bind-address            = *
mysqlx-bind-address     = *
```

Da åpner du for for alle IP-adresser. Både IPv4 oggit push -u origin branch
IPv6.

### Åpne for flere samtidige tilkoblinger

podcast.srib.no hadde et problem med at nye podcast-episoder ikke dukket opp på
ulike tredjepart-plattformer. Feilmeldingen som dukket opp i Django-koden var:
`MySQLdb._exceptions.OperationalError: (1040, 'Too many connections')`.

Dette betyr at MySQL-tjeneren ikke tillater nok samtidige tilkoblinger, og
nekter dermed alle andre enn de XXX første tilkoblingene.

For å fikse dette, gjør en følgende:

1. Inne på VM-en: `sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf`.
2. Finn `max_connections = XXXX`.
3. Endre `max_connections` til å være noe mellom 1 og 10'000. (jo høyere tall,
   jo flere tilkoblinger tillater du).
4. Lagre filen og restart MySQL-tjenesten med `sudo service mysql restart`.

Om du får en feilmelding på steg 4, er det godt mulig at det er en syntaks-feil
i `/etc/mysql/mysql.conf.d/mysqld.cnf`.

For å sjekke om forandringen har manifestert seg, kan du skrive
`show variables like "max_connections";` etter at du har logget inn i
MySQL-tjeneren.

Alternativt, kan du også skrive `set global max_connections = 5000;` mens du er
logget inn i MySQL-tjeneren

### Kilder:

[MySQL: Error Code: 1118 Row size too large (> 8126).](https://stackoverflow.com/a/37076100),
sist lest 06.03.23

[How to fix ERROR 1726 (HY000): Storage engine 'MyISAM' does not support system tables](https://stackoverflow.com/a/55838493),
sist lest 06.03.23

[Change root password](https://stackoverflow.com/a/58964143), sist lest 06.03.23

[Mysql 8 bind-address syntax](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_bind_address),
sist lest 06.03.23

[How to increase MySQL connections(max_connections)](https://stackoverflow.com/a/22297829),
sist lest 21.03.23

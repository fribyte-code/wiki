+++
title = "Database Migration"
description = "An explanation of how I migrated all the databases from Donald (Ubuntu 14.04) to a VM on Konrad (Ubuntu 22.02)."
template = "docs/page.html"
sort_by = "weight"
weight = 10003

[extra]
lang = "en"
translation = "docs/instrukser/migrere-database.md"
+++

friByte hosts SQL databases for clients. This is something we have more or less
always done, but the new databases we host are located on the VMs in ProxMox.
Previously, all databases were on a machine server named Donald. Since many of
the former clients who actively used those databases no longer use them, the
database had fallen into disrepair.

We therefore recently (06.03.2023) moved all the databases over to a single VM
in the ProxMox cluster, with the intuitive name "database". In this way we
ensure regular backups while also modernizing the database.

### Transfer the databases

1. On Donald: `mysqldump --all-databases --replace --events > donald.sql`
2. On Donald:
   `rsync -r -v --info=progress2 donald.sql root@158.37.6.28:/root`
3. On Konrad:
   `rsync -r -v --info=progress2 /root/donald.sql fribyte@158.37.6.37:/home/fribyte`
4. On the VM: `sudo apt install pv`\*
5. In MySQL on the VM: `SET GLOBAL innodb_strict_mode=OFF;`
6. On the VM: `pv donald.sql | sudo mysql -u root -p`
7. On the VM: `sudo service mysql stop`
8. On the VM: `sudo mysqld --upgrade=FORCE`
9. On the VM: `sudo service mysql start`

\*_"pv" stands for "pipeviewer" and is only there so you can see the progress of
the transfer._

Once all of this was completed, the login information and user information from
the SQL dump were transferred into the fresh MySQL server. Which means that you
can log in to this database with the same users as the old database.

But the permissions did not work quite properly, so I also had to change the
root password to get full admin access.

### How I changed the root password

1. On the VM: `sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf` and add
   `skip-grant-tables` under `[mysqld]`. Save and close.
2. On the VM: `sudo service mysl restart`
3. On the VM: `sudo mysql -u root`
4. In MySQL on the VM: `FLUSH PRIVILEGES;`
5. In MySQL on the VM:
   `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'lololol';`

If you now remove `skip-grant-tables` from `/etc/mysql/mysql.conf.d/mysqld.cnf`,
and run `sudo service mysql restart`, you can now log in as `root` with
password: `lololol`.

After this, I was able to change the passwords for the users that came with
`donald.sql`. I did the following:

1. On the VM: `sudo mysql -u -p` _enter password_
2. In MySQL on the VM:
   `ALTER USER 'digas'@'%' IDENTIFIED WITH mysql_native_password BY 'lololol';`\*\*
3. In MySQL on the VM: `FLUSH PRIVILEGES;`

\*\*_"digas" is the database user for SRiB's DigaSystem program. "%" means any
host_

### Full disk during transfer

This process took some time, and required quite a bit more storage space on the
VM than I expected. So the transfer crashed several times due to lack of space.

The SQL dump, donald.sql, used 5.1 GB of storage, but when it was transferred to
the MySQL server, the database used 12 GB. So, 17.1 GB in total.

So this is how I handled a full disk.

1. On the VM:
   `sudo apt-get purge mysql-server mysql-client mysql-common mysql-server-core-* mysql-client-core-* && sudo rm -rf /etc/mysql /var/lib/mysql`
2. On the VM: `sudo apt install mysql-server -y`
3. In MySQL: `SET GLOBAL innodb_strict_mode=OFF;`
4. Assign more storage space to the VM. This can be done in the ProxMox web
   GUI.
5. On the VM: `pv donald.sql | sudo mysql -u root -p`

### Open for external connections

For external machines to be able to connect, we need to put this into the
configuration.

1. On the VM: `sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf`
2. Change:

```bash
bind-address            = 127.0.0.1
mysqlx-bind-address     = 127.0.0.1
```

to:

```bash
bind-address            = 0.0.0.0
mysqlx-bind-address     = 0.0.0.0
```

This only opens for all IPv4 addresses. Alternatively, you can also change it
to:

```bash
bind-address            = *
mysqlx-bind-address     = *
```

This opens for all IP addresses. Both IPv4 and IPv6.

### Open for more simultaneous connections

`podcast.srib.no` had a problem where new podcast episodes did not appear on
various third-party platforms. The error message that appeared in the Django
code was: `MySQLdb._exceptions.OperationalError: (1040, 'Too many connections')`.

This means that the MySQL server does not allow enough simultaneous
connections, and therefore rejects all connections beyond the first XXX
connections.

To fix this, do the following:

1. On the VM: `sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf`.
2. Find `max_connections = XXXX`.
3. Change `max_connections` to something between 1 and 10'000. (the higher the
   number, the more connections you allow).
4. Save the file and restart the MySQL service with
   `sudo service mysql restart`.

If you get an error message at step 4, it is quite possible that there is a
syntax error in `/etc/mysql/mysql.conf.d/mysqld.cnf`.

To check whether the change has taken effect, you can run
`show variables like "max_connections";` after logging in to the MySQL server.

Alternatively, you can also run `set global max_connections = 5000;` while
logged in to the MySQL server

### Sources

[MySQL: Error Code: 1118 Row size too large (> 8126).](https://stackoverflow.com/a/37076100),
last read 06.03.23

[How to fix ERROR 1726 (HY000): Storage engine 'MyISAM' does not support system tables](https://stackoverflow.com/a/55838493),
last read 06.03.23

[Change root password](https://stackoverflow.com/a/58964143), last read 06.03.23

[Mysql 8 bind-address syntax](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_bind_address),
last read 06.03.23

[How to increase MySQL connections(max_connections)](https://stackoverflow.com/a/22297829),
last read 21.03.23

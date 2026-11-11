# Zabbix8_on_Debian13


## :book: Index

* 👮 [Terms of use](./LICENSE)
* ♻️ [Features](#recycle-features)
* ✅ [Requirements](#white_check_mark-requirements)
* 📚 [Resources](#books-resources)
* ⚙️ [Install basic software](#gear-install-basic-software)
* 🧰 [Configure PostgreSQL](#toolbox-configure-postgresql)
* 🕛 [Configure TimeScaleDB](#clock12-configure-timescaledb)
* 🛠️ [Configure Zabbix server](#hammer_and_wrench-configure-zabbix-server)
* 🖥️ [Configure Nginx for Zabbix frontend](#desktop_computer-configure-nginx-for-zabbix-frontend)
* 🔗 [Final web server configuration](#link-final-web-server-configuration)
* ⚔️ [Login](#crossed_swords-login)
* 🔭 [Front](#telescope-front)

## :recycle: Features

- Compatible with any Debian-based distribution
- The most powerful build you can make of Zabbix
- PostgreSQL is among the top databases
- TimeScaleDB will speed up PostgreSQL and compress the database
- Nginx is among the top web servers
- Database installation on a separate disk
- Guide 100% compatible with production environments

## :white_check_mark: Requirements

- Basic knowledge of Linux
- Basic knowledge of databases
- One 10GB disk minimal for Debian and another 10GB disk dedicated for PostgreSQL data 
- From 4GB to 16GB of RAM, depending on the environment
- Between 2 and 4 CPU cores, depending on the environment
- Internet connection to install software

## :books: Resources:
-  PostgreSQL DB: https://www.postgresql.org/docs/
-  Zabbix: https://www.zabbix.com/manuals
- TimeScaleDB: https://www.tigerdata.com/docs
- Nginx: https://nginx.org/en/docs/

## :gear: Install basic software 

### Install basic system packages:
```bash
apt update
apt install rsync sudo gnupg curl locales iproute2
```

Add our system user to sudo:
```bash
usermod -aG sudo YOUR_USER
```

👁️ Now that our normal user has the necessary permissions to carry everything out, we will log out from the root account. If we had a session started with our normal user, we will also log out and log back in so that the sudo group permissions take effect. From this point on, we will no longer use the root user throughout the tutorial

---
### Install PostgreSQL

Debian13 uses PostgreSQL17 by default. If you use another Linux distribution or install the PostgreSQL repositories, the version may be 18. In our guide, stability takes priority over everything else, and this is what Debian recommends
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```
## :toolbox: Configure PostgreSQL 

### Change the database directory to another disk
This step is optional. We recommend separating the database onto another disk for security, management, and performance reasons. By being on a separate disk, it won’t perform writes on the system disk, and if it fills up, it won’t bring the Debian system down.

We have previously created a 20GB disk and mounted it as ext4 at /mnt/storedb

❌ If you don’t want to change the PostgreSQL directory, you can safely skip this step
```bash
sudo systemctl stop postgresql
sudo mkdir -p /mnt/storedb/postgresql/17/main/ # Or the custom path where you want the database to live
sudo rsync -av /var/lib/postgresql/17/main/ /mnt/storedb/postgresql/17/main/
sudo chown -R postgres:postgres /mnt/storedb/postgresql
```
Edit data directory:
```bash
 sudo nano /etc/postgresql/17/main/postgresql.conf
```
```bash
data_directory = '/mnt/storedb/postgresql/17/main'  # use data in another directory
```
Start database service:
```bash
sudo systemctl start postgresql
```
### Test new database config:
When executing the command, the new path we created for the database directory should be reflected.
```bash
sudo pg_lsclusters
```
```bash
Ver Cluster Port Status Owner    Data directory              Log file
17  main    5432 online postgres /mnt/storedb/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
```
### Configure database acces:
In most step-by-step guides, the credentials are usually zabbix / zabbix. This often creates confusion between the system user, the database user, or even within the command syntax that follows. We have changed the names for a much more secure installation and one that is easier to understand

Create user usrzabbixdb:
```bash
sudo -u postgres createuser --pwprompt usrzabbixdb
```
⚠️ Save the password you set for later use

Create database datazabbixdb:
```bash
sudo -u postgres createdb -O usrzabbixdb datazabbixdb
```

### Install Zabbix server, frontend, Zabbix agent 2, Nginx 

Add Zabbix repository:
```bash
wget https://repo.zabbix.com/zabbix/8.0/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_8.0+debian13_all.deb
sudo dpkg -i zabbix-release_latest_8.0+debian13_all.deb
sudo apt update
```

Install packages:
```bash
sudo apt install zabbix-server-pgsql zabbix-frontend-php php8.4-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent2 nginx
```

Install Postgresql Plugin:
```bash
 apt install zabbix-agent2-plugin-postgresql
```

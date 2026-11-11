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

### Install system packages:
```bash
apt update
apt install rsync sudo gnupg curl locales iproute2
```

Add our system user to sudo:
```bash
usermod -aG sudo YOUR_USER
```

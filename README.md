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
apt install rsync sudo gnupg curl locales
```

👁️ Now that our normal user has the necessary permissions to carry everything out, we will log out from the root account. If we had a session started with our normal user, we will also log out and log back in so that the sudo group permissions take effect. From this point on, we will no longer use the root user throughout the tutorial

---
### Install PostgreSQL

Debian13 uses PostgreSQL17 by default. If you use another Linux distribution or install the PostgreSQL repositories, the version may be 18. In our guide, stability takes priority over everything else, and this is what Debian recommends
```bash
sudo apt update
sudo apt install postgresql
```
## :toolbox: Configure PostgreSQL 

### Change the database directory to another disk
This step is optional. We recommend separating the database onto another disk for security, management, and performance reasons. By being on a separate disk, it won’t perform writes on the system disk, and if it fills up, it won’t bring the Debian system down.

We have previously created a 20GB disk and mounted it as ext4 at /mnt/storedb

❌ If you don’t want to change the PostgreSQL directory, you can safely skip this step
```bash
sudo systemctl stop postgresql
sudo mkdir -p /mnt/storedb/postgresql/17/main/ # Or the custom path where you want the database to live
sudo rsync -av /var/lib/postgresql/17/main/ /mnt/storedb/postgresql/17/main/ # Or the custom path where you want the database to live
sudo chown -R postgres:postgres /mnt/storedb/postgresql # Or the custom path where you want the database to live
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

### Configure database access:
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
sudo apt install zabbix-server-pgsql zabbix-frontend-php php8.4-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent2 zabbix-agent2-plugin-postgresql nginx
```
### Install TimeScaleDB plugin:
TimescaleDB is an extension for PostgreSQL that transforms it into a database optimized for time-series data. For Zabbix, it works by automatically splitting large history tables into time-based ‘chunks’, which are invisible to the user but extremely efficient for the system.

Goodbye to slowdowns (efficient Housekeeper): Instead of deleting records one by one (which slows down the server), TimescaleDB removes entire time chunks instantly and with virtually zero CPU cost.

Significant disk savings: It enables native compression that reduces the storage used by historical data by between 60% and 90%.
Consistent performance: It prevents Zabbix from slowing down over time. Read and write speeds for graphs remain fast regardless of database size.

Using TimescaleDB is essential if you are going to monitor a medium or large environment, as it prevents the most common Zabbix bottleneck: database saturation.

Download the repository keys:
```bash
sudo curl -fSL https://packagecloud.io/timescale/timescaledb/gpgkey \
| sudo gpg --dearmor -o /usr/share/keyrings/timescaledb.gpg
```
Install the repository of TimeScaleDB:
```bash
echo "deb [signed-by=/usr/share/keyrings/timescaledb.gpg] https://packagecloud.io/timescale/timescaledb/debian/ trixie main" \
| sudo tee /etc/apt/sources.list.d/timescaledb.list
```
Refresh APT:
```bash
sudo apt update
```
Install TimeScaleDB from APT:
```bash
  apt install timescaledb-2-2.29.2-postgresql-17  timescaledb-2-loader-postgresql-17=2.29.2~debian13-1711
```
## :clock12: Configure TimeScaleDB
Now, with all the software installed on the server, we proceed to its configuration so that all the components fit together perfectly.

### Tune TimeScale for PostgreSQL:
The command sudo timescaledb-tune --quiet --yes is an automatic optimization tool that analyzes your server resources (such as RAM and CPU) and automatically adjusts the parameters of the postgresql.conf file
```bash
sudo timescaledb-tune --quiet --yes
```
Restart database service:
```bash
sudo systemctl restart postgresql
```
Activate TimeScaleDB on datazabbixdb:
```bash
sudo -u postgres psql -d datazabbixdb -c "CREATE EXTENSION IF NOT EXISTS timescaledb;"
```
✔️ CREATE EXTENSION

### Import schema database
```bash
zcat /usr/share/zabbix/sql-scripts/postgresql/server.sql.gz | sudo -u postgres psql -d datazabbixdb
```

### Execute schema optimization TimeScaleDB:

```bash
 sudo -u postgres psql -d dbzabbix -f /usr/share/zabbix/sql-scripts/postgresql/timescaledb/schema.sql
```

❌ If you missed any of these steps or something went wrong, the best approach is to drop the database and recreate it following the previous steps
```bash
sudo -u postgres dropdb datazabbixdb
```

Test TimeScaleDB:
This test confirms that TimescaleDB is working in Zabbix because it verifies that the historical tables (such as history or trends) have been converted into hypertables, which is TimescaleDB’s special format for handling time-series data. If these tables appear in the query results, it means the extension is active, properly integrated into the database, and managing automatic time-based partitioning.

```bash
 sudo -u postgres psql -d datazabbixdb -c "SELECT hypertable_name FROM timescaledb_information.hypertables;"
```
Correct result:
```bash
  hypertable_name
-----------------
 history
 history_uint
 history_log
 history_text
 history_str
 history_bin
 auditlog
 trends
 trends_uint
(9 filas)
```
### Skip TimescaleDB version restriction:
⚠️ This step is only necessary if you have installed software versions that Zabbix does not support by default. Use it at your own risk; we include it so you are aware that it exists.
Uncomment and set the AllowUnsupportedDBVersions parameter to 1:
```bash
sudo nano /etc/zabbix/zabbix_server.conf
```
```bash
### Option: AllowUnsupportedDBVersions
#       Allow server to work with unsupported database versions.
#       0 - do not allow
#       1 - allow
#
# Mandatory: no
# Default:
AllowUnsupportedDBVersions=1
```
## :hammer_and_wrench: Configure Zabbix server:
### Configure the database credentials for zabbix server:
Change parameters, in this file, we will add the password for the PostgreSQL user usrzabbixdb that we created earlier
```bash
sudo nano /etc/zabbix/zabbix_server.conf
```
```bash
### Option: DBName
#       Database name.
#       If the Net Service Name connection method is used to connect to Oracle database, specify the service name from
#       the tnsnames.ora file or set to empty string; also see the TWO_TASK environment variable if DBName is set to
#       empty string.
#
# Mandatory: yes
# Default:
# DBName=

DBName=datazabbixdb

### Option: DBSchema
#       Schema name. Used for PostgreSQL.
#
# Mandatory: no
# Default:
# DBSchema=

### Option: DBUser
#       Database user.
#
# Mandatory: no
# Default:
# DBUser=

DBUser=usrzabbixdb

### Option: DBPassword
#       Database password.
#       Comment this line if no password is used.
#
# Mandatory: no
# Default:
DBPassword=YOUR_PASSWORD_DB_USER
```
### Enable global scripts:
This will enable commands such as right-clicking on hosts and using the ping button
```bash
sudo nano /etc/zabbix/zabbix_server.conf
```
```bash
### Option: EnableGlobalScripts
#    Enable global scripts on Zabbix server.
#       0 - disable
#       1 - enable
#
# Mandatory: no
# Default:
EnableGlobalScripts=1
# EnableGlobalScripts=0
```

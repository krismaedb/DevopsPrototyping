Using this resources for installing zabbix
https://www.zabbix.com/download?zabbix=6.4&os_distribution=ubuntu&os_version=22.04&components=server_frontend_agent&db=mysql&ws=apache

Steps for TimescaleDB on Ubuntu 22.04 + PostgreSQL 14
Add official TimescaleDB repo
sudo sh -c "echo 'deb https://packagecloud.io/timescale/timescaledb/ubuntu/ jammy main' > /etc/apt/sources.list.d/timescaledb.list"

 Add the GPG key
 curl -L https://packagecloud.io/timescale/timescaledb/gpgkey | sudo apt-key add -
sudo apt update

 Install TimescaleDB for PostgreSQL 14
 sudo apt install timescaledb-2-postgresql-14

 After installation, check extension
 sudo -u postgres psql zabbix -c "\dx"

Expected output should show:
timescaledb   | 2.x.x | public

Schema created
CREATE SCHEMA zabbix AUTHORIZATION zabbixuser;

1. Verify Existing Zabbix Database
sudo -u postgres psql -c "\l"
Ensure that the zabbix database exists and note the owner (ex: zabbixuser).

2. Create Zabbix Schema
sudo -u postgres psql -d zabbix -c "CREATE SCHEMA zabbix AUTHORIZATION zabbixuser;"

4. Add TimescaleDB Repository Key
wget -qO- https://packagecloud.io/timescale/timescaledb/gpgkey \
| sudo tee /etc/apt/trusted.gpg.d/timescaledb.asc

6. Update Package List
sudo apt-get update

8. Install TimescaleDB Tuning Tool
(Used to automatically update postgresql.conf)
sudo apt install timescaledb-tools

6. Run TimescaleDB Tuner
sudo timescaledb-tune
Choose:
Yes for updating shared_preload_libraries
Yes for memory settings
Yes for parallelism
Save changes to /etc/postgresql/14/main/postgresql.conf
This adds:
shared_preload_libraries = 'timescaledb'

7. Restart PostgreSQL
sudo systemctl restart postgresql

9. Verify Preload Library
sudo -u postgres psql -c "SHOW shared_preload_libraries;"
Expected output:
 timescaledb

9. Create TimescaleDB Extension in Zabbix Schema
sudo -u postgres psql -d zabbix -c \
"CREATE EXTENSION IF NOT EXISTS timescaledb WITH SCHEMA zabbix CASCADE;"
10. Verify Extension Installation
sudo -u postgres psql -d zabbix -c "\dx"
Expected:

plpgsql     | 1.0     | pg_catalog
timescaledb | 2.xx    | zabbix
Check schemas:
sudo -u postgres psql -d zabbix -c "\dn"

Schemas should include:
_timescaledb_*
timescaledb_information
zabbix

11. Confirm Extension Metadata
sudo -u postgres psql zabbix -c \
"SELECT extname, extversion FROM pg_extension;"

Should show:
plpgsql
timescaledb


Step 8: Configure Zabbix Server
bash# Edit Zabbix server config
nano /etc/zabbix/zabbix_server.conf
Find and modify these lines:
ini# Database settings
DBName=zabbix
DBUser=zabbixuser
DBPassword=CTaylorSwift13
# Uncomment and set (around line 100-150)
DBHost=localhost

Step 9: Enable the Zabbix Apache configuration
Run:
sudo a2enconf zabbix

Then reload/restart Apache:
sudo systemctl reload apache2
sudo systemctl restart apache2

Step 10: Verify PHP settings
Open the Zabbix Apache config file:
sudo nano /etc/apache2/conf-available/zabbix.conf

Inside you should see something similar to:

<Directory "/usr/share/zabbix">
    php_value date.timezone America/Winnipeg
</Directory>

If not, add the timezone line manually.
Save → exit → restart Apache again:

sudo systemctl restart apache2


Step 11: Activate Firewall (UFW)
sudo ufw status

If it's active and blocking port 80:
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# PART 2: INITIAL SYSTEM SETUP
#-----------------------------------------------------------------------------
# Login as zadmin, switch to root
sudo -i

# Update system
apt update && apt upgrade -y

# Install basic tools
apt install -y vim wget curl net-tools htop

# Set timezone
timedatectl set-timezone America/Winnipeg

# Verify network configuration
ip addr show
ping -c 4 10.10.40.1    # Gateway
ping -c 4 10.10.40.10   # DC01
ping -c 4 8.8.8.8       # Internet

# PART 3: INSTALL POSTGRESQL DATABASE
#-----------------------------------------------------------------------------
# Install PostgreSQL 14
apt install -y postgresql postgresql-contrib

# Start and enable PostgreSQL service
systemctl start postgresql
systemctl enable postgresql
systemctl status postgresql

#-----------------------------------------------------------------------------
# PART 4: CREATE ZABBIX DATABASE AND USER
#-----------------------------------------------------------------------------
# Connect to PostgreSQL as postgres user
sudo -u postgres psql

# Execute these SQL commands in PostgreSQL prompt:
DROP DATABASE IF EXISTS zabbix;
CREATE DATABASE zabbix ENCODING 'UTF8' LC_COLLATE 'en_US.UTF-8' LC_CTYPE 'en_US.UTF-8' TEMPLATE template0;
CREATE USER zabbixuser WITH PASSWORD 'g3company!@#';
ALTER DATABASE zabbix OWNER TO zabbixuser;
GRANT ALL PRIVILEGES ON DATABASE zabbix TO zabbixuser;
\c zabbix
GRANT ALL ON SCHEMA public TO zabbixuser;
\q

#-----------------------------------------------------------------------------
# PART 5: INSTALL ZABBIX SERVER AND COMPONENTS
#-----------------------------------------------------------------------------
# Download Zabbix 7.4 repository package
wget https://repo.zabbix.com/zabbix/7.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.4-1+ubuntu22.04_all.deb

# Install Zabbix repository
dpkg -i zabbix-release_7.4-1+ubuntu22.04_all.deb

# Update package index
apt update

# Install Zabbix server, frontend, agent, and SQL scripts
apt install -y zabbix-server-pgsql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent

# Verify Zabbix packages are installed
dpkg -l | grep zabbix

#-----------------------------------------------------------------------------
# PART 6: DOWNLOAD AND IMPORT DATABASE SCHEMA
#-----------------------------------------------------------------------------
# Download Zabbix 7.4.5 source code (contains SQL schema files)
cd /tmp
wget https://cdn.zabbix.com/zabbix/sources/stable/7.4/zabbix-7.4.5.tar.gz

# Extract the archive
tar -xzf zabbix-7.4.5.tar.gz

# Navigate to PostgreSQL database directory
cd zabbix-7.4.5/database/postgresql/

# Import database schema as zabbixuser (to create tables in public schema)
psql -h localhost -U zabbixuser -d zabbix -W < schema.sql
# Password when prompted: g3company!@#

# Import images
psql -h localhost -U zabbixuser -d zabbix -W < images.sql
# Password when prompted: g3company!@#

# Import initial data
psql -h localhost -U zabbixuser -d zabbix -W < data.sql
# Password when prompted: g3company!@#

# Verify tables were created in public schema
sudo -u postgres psql -d zabbix -c "SELECT COUNT(*) FROM information_schema.tables WHERE table_schema = 'public';"
# Should return: 207 tables

# Verify dbversion table exists
sudo -u postgres psql -d zabbix -c "SELECT * FROM public.dbversion;"
# Should return: dbversionid=1, mandatory=7040000, optional=7040000

#-----------------------------------------------------------------------------
# PART 7: GRANT DATABASE PERMISSIONS
#-----------------------------------------------------------------------------
# Connect to database and grant all necessary permissions
sudo -u postgres psql -d zabbix

# Execute these SQL commands:
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO zabbixuser;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO zabbixuser;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO zabbixuser;
\q

#-----------------------------------------------------------------------------
# PART 8: CONFIGURE POSTGRESQL AUTHENTICATION
#-----------------------------------------------------------------------------
# Edit PostgreSQL host-based authentication file
nano /etc/postgresql/14/main/pg_hba.conf

# Modify these lines (change 'peer' and 'scram-sha-256' to 'md5'):
# BEFORE:
# local   all             all                                     peer
# host    all             all             127.0.0.1/32            scram-sha-256
# host    all             all             ::1/128                 scram-sha-256
#
# AFTER:
# local   all             all                                     md5
# host    all             all             127.0.0.1/32            md5
# host    all             all             ::1/128                 md5

# Save file (Ctrl+X, Y, Enter)

# Restart PostgreSQL to apply changes
systemctl restart postgresql

#-----------------------------------------------------------------------------
# PART 9: CONFIGURE ZABBIX SERVER
#-----------------------------------------------------------------------------
# Edit Zabbix server configuration file
nano /etc/zabbix/zabbix_server.conf

# Find and modify/uncomment these lines:
# DBHost=localhost
# DBName=zabbix
# DBUser=zabbixuser
# DBPassword=g3company!@#
#
# Note: Remove quotes around password if present

# Save file (Ctrl+X, Y, Enter)

# Verify configuration is correct
grep -v "^#" /etc/zabbix/zabbix_server.conf | grep -v "^$" | grep DB

# Should show:
# DBHost=localhost
# DBName=zabbix
# DBUser=zabbixuser
# DBPassword=g3company!@#

# Test Zabbix server configuration
zabbix_server -c /etc/zabbix/zabbix_server.conf -T
# Should output: "Validation successful"

#-----------------------------------------------------------------------------
# PART 10: CONFIGURE PHP TIMEZONE FOR ZABBIX
#-----------------------------------------------------------------------------
# Edit Zabbix Apache configuration
nano /etc/zabbix/apache.conf

# Verify this line exists (uncomment if needed):
# php_value date.timezone America/Winnipeg

# Save file

#-----------------------------------------------------------------------------
# PART 11: START AND ENABLE ZABBIX SERVICES
#-----------------------------------------------------------------------------
# Start Zabbix server
systemctl start zabbix-server

# Enable Zabbix server to start on boot
systemctl enable zabbix-server

# Check Zabbix server status
systemctl status zabbix-server
# Should show: active (running)

# Start and enable Zabbix agent
systemctl start zabbix-agent
systemctl enable zabbix-agent

# Restart Apache web server
systemctl restart apache2
systemctl enable apache2

#-----------------------------------------------------------------------------
# PART 12: VERIFY INSTALLATION
#-----------------------------------------------------------------------------
# Check Zabbix server is running
systemctl status zabbix-server

# Check Zabbix is listening on port 10051
ss -tunlp | grep 10051

# Check Apache is serving Zabbix web interface
ss -tunlp | grep :80

# Test database connection manually
psql -h localhost -U zabbixuser -d zabbix -W -c "SELECT * FROM dbversion;"
# Password: g3company!@#
# Should return version information

#-----------------------------------------------------------------------------
# PART 13: ACCESS ZABBIX WEB INTERFACE
#-----------------------------------------------------------------------------
# Open web browser and navigate to:
# http://10.10.40.40/zabbix

# Complete Setup Wizard:
# 1. Welcome page → Click "Next step"
# 2. Check of pre-requisites → Verify all green → Click "Next step"
# 3. Configure DB connection:
#    - Database type: PostgreSQL
#    - Database host: localhost
#    - Database port: 5432
#    - Database name: zabbix
#    - Database schema: (leave empty)
#    - Store credentials in: Plain text
#    - User: zabbixuser
#    - Password: g3company!@#
#    → Click "Next step"
# 4. Zabbix server details:
#    - Host: localhost
#    - Port: 10051
#    - Name: Healthcare Clinic Zabbix
#    → Click "Next step"
# 5. Pre-installation summary → Review → Click "Next step"
# 6. Congratulations page → Click "Finish"

# Login to Zabbix:
# - Username: Admin (capital A)
# - Password: zabbix

# IMPORTANT: Change admin password immediately:
# 1. Administration → Users
# 2. Click "Admin"
# 3. Click "Change password"
# 4. New password: Clinic@2025!
# 5. Confirm password: Clinic@2025!
# 6. Click "Update"

#-----------------------------------------------------------------------------
# TROUBLESHOOTING COMMANDS (if needed)
#-----------------------------------------------------------------------------
# View Zabbix server logs
tail -50 /var/log/zabbix/zabbix_server.log

# View systemd journal for Zabbix
journalctl -xeu zabbix-server.service -n 50

# Restart all services
systemctl restart postgresql
systemctl restart zabbix-server
systemctl restart zabbix-agent
systemctl restart apache2

# Check PostgreSQL is accepting connections
ss -tunlp | grep 5432

# Test database connectivity
psql -h localhost -U zabbixuser -d zabbix -W

# Check Zabbix server configuration syntax
zabbix_server -c /etc/zabbix/zabbix_server.conf -T

#=============================================================================
# CREDENTIALS SUMMARY
#=============================================================================
# PostgreSQL Database:
#   - Superuser: postgres
#   - Superuser Password: TaylorSwift13
#   - Database Name: zabbix
#   - Database Owner: zabbixuser
#   - Zabbix User: zabbixuser
#   - Zabbix User Password: g3company!@#
#
# Zabbix Web Interface:
#   - URL: http://10.10.40.40/zabbix
#   - Default Username: Admin
#   - Default Password: zabbix
#   - New Password (after change): Clinic@2025!
#
# Operating System:
#   - Hostname: zabbix01
#   - IP Address: 10.10.40.40/24
#   - Gateway: 10.10.40.1
#   - DNS: 10.10.40.10, 10.10.40.11
#   - System User: zadmin
#   - System Password: Clinic@2025!
#
#=============================================================================
# NETWORK INFORMATION
#=============================================================================
# VLAN: 40 (IT/Servers)
# IP Address: 10.10.40.40/24
# Gateway (HSRP VIP): 10.10.40.1
# DNS Servers: 
#   - Primary: 10.10.40.10 (DC01)
#   - Secondary: 10.10.40.11 (DC02)
# Domain: healthclinic.local
#
# Connected to: SW1 Port G0/10 (VLAN 40)
#
#=============================================================================
# SERVICES AND PORTS
#=============================================================================
Zabbix Server: Port 10051 (listening)
Zabbix Agent: Port 10050 (listening)
Apache Web Server: Port 80 (HTTP)
PostgreSQL Database: Port 5432 (localhost only)


# KEY TROUBLESHOOTING NOTES

Issue 1: "table dbversion was not found"
- Cause: Tables were created in wrong schema (not 'public')
- Solution: Import schema as zabbixuser (not postgres user)

Issue 2: "missing mandatory parameter DBName"
- Cause: DBHost typo (localost instead of localhost)
- Solution: Fix typo in /etc/zabbix/zabbix_server.conf

Issue 3: "Cannot connect to database"
- Cause: PostgreSQL authentication method (peer/scram-sha-256)
- Solution: Change to md5 in pg_hba.conf



# TROUBLESHOOTING COMMANDS (if needed)

View Zabbix server logs
tail -50 /var/log/zabbix/zabbix_server.log

View systemd journal for Zabbix
journalctl -xeu zabbix-server.service -n 50

Restart all services
systemctl restart postgresql
systemctl restart zabbix-server
systemctl restart zabbix-agent
systemctl restart apache2

Check PostgreSQL is accepting connections
ss -tunlp | grep 5432

Test database connectivity
psql -h localhost -U zabbixuser -d zabbix -W

Check Zabbix server configuration syntax
zabbix_server -c /etc/zabbix/zabbix_server.conf -T


# CREDENTIALS SUMMARY

PostgreSQL Database:
- Superuser: postgres
- Superuser Password: TaylorSwift13
- Database Name: zabbix
- Database Owner: zabbixuser
- Zabbix User: zabbixuser
- Zabbix User Password: g3company!@#

Zabbix Web Interface:
- URL: http://10.10.40.40/zabbix
- Default Username: Admin
- Default Password: zabbix


# PART 14: ADD NETWORK DEVICES TO ZABBIX
🐧 ADD WEB01 (Web Server) TO ZABBIX

Step 1: Download Correct Repository
Still on WEB01
cd /tmp

 CORRECT COMMANDS FOR ZABBIX 7.0 LTS + UBUNTU 24.04
On WEB01

1. Download Zabbix repository
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb

2. Install repository package
sudo dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb

3. Update package list
sudo apt update

4. Install Zabbix Agent 2
sudo apt install -y zabbix-agent2

5. Verify installation
dpkg -l | grep zabbix


Step 3: Configure Agent - CRITICAL PART

Edit configuration
sudo nano /etc/zabbix/zabbix_agent2.conf

Find and modify EXACTLY these 3 lines:
Press Ctrl+W to search for each:

Search: Server=
ini### Option: Server
Server=10.10.40.40

Search: ServerActive=

ini### Option: ServerActive
ServerActive=10.10.40.40

Search: Hostname=
ini### Option: Hostname
Hostname=WEB01

IMPORTANT: Make sure to uncomment (remove # if present)!
Save: Ctrl+X → Y → Enter

Step 8: Verify Configuration

bash# Check configuration
sudo grep -E "^(Server|ServerActive|Hostname)" /etc/zabbix/zabbix_agent2.conf

Should show EXACTLY:
Server=10.10.40.40
ServerActive=10.10.40.40
Hostname=WEB01

Step 4: Start Zabbix Agent
Start agent
sudo systemctl start zabbix-agent2

Enable on boot
sudo systemctl enable zabbix-agent2

Check status
sudo systemctl status zabbix-agent2

Should show: active (running) ✅


Step 5: Verify Port is Listening
Check port 10050
sudo ss -tunlp | grep 10050

Should show:
LISTEN  0  128  0.0.0.0:10050  users:(("zabbix_agent2",pid=XXXX))

Step 5: Test from ZABBIX01
bash# Exit from WEB01
exit

Now on ZABBIX01, test connectivity
zabbix_get -s 10.10.40.30 -k agent.ping

Should return: 1 ✅

zabbix_get -s 10.10.40.30 -k agent.version


# ADD IN ZABBIX WEB UI
Host name: WEB01
IP: 10.10.40.30
Template: Linux by Zabbix agent
Host group: Linux servers


Step 2: Create Host

Go to: Data collection → Hosts
Click "Create host" (top right)


Step 3: Fill in Host Details
Host tab:
FieldValueHost name - WEB01 ← EXACTLY "WEB01", nothing else!
Visible name - WEB01 - Web Application Server
Templates - Click "Select" → Search: Linux → Check ☑️ "Linux by Zabbix agent" → Click "Select"
Host groups - Click "Select" → Check ☑️ "Linux servers" → Click "Select"

Interfaces section:
1. Click "Add" → Select "Agent"
Fill in:
IP address: 10.10.40.30
DNS name: (leave empty)
Connect to: IP
Port: 10050




Step 4: Save
Click "Add" (blue button at bottom)
Should see: "Host added" success message

Step 5: Verify Status
Go to: Monitoring → Hosts
Find WEB01
Wait 2-3 minutes (important!)
Refresh page: Press Ctrl+F5

Should see:
Status: Enabled (green)
Availability: 🟢 ZBX (green icon)


Step 6: Check Latest Data
On Monitoring → Hosts page
Click "Latest data" link next to WEB01

Should see metrics:
CPU usage
Memory usage
Disk space
Network traffic
System uptime






































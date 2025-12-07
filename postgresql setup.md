# PostgreSQL + pgAdmin (Docker) Remote Connection SOP

## 1. Enable SSH on the Server
sudo systemctl enable ssh
sudo systemctl start ssh

## 2. Check Server Network Interfaces
ip a
# Confirm server IP (e.g., 10.10.40.30)

## 3. Ensure PostgreSQL is Running
sudo systemctl status postgresql
sudo systemctl start postgresql

## 4. Configure PostgreSQL to Accept Remote Connections
sudo nano /etc/postgresql/*/main/postgresql.conf
# Change
#listen_addresses = 'localhost'
# to
listen_addresses = '*'

## 5. Create PostgreSQL User with Login
sudo -u postgres psql
# Inside psql shell:
CREATE ROLE webadmin WITH LOGIN PASSWORD 'T@ylorSwift13';
ALTER ROLE webadmin WITH SUPERUSER;
\q
# Verify:
sudo -u postgres psql -c "\du"

## 6. Configure pg_hba.conf for Remote Access
sudo nano /etc/postgresql/*/main/pg_hba.conf
# Add at bottom:
host    all    all    10.10.40.0/24    md5
host    all    all    172.17.0.0/16    md5
# Optional (testing only):
host    all    all    0.0.0.0/0    md5
sudo systemctl restart postgresql

## 7. Test Remote Login via CLI
psql -h 10.10.40.30 -U webadmin -d postgres
# Enter password 'T@ylorSwift13'

## 8. pgAdmin Docker GUI Connection Settings
# In pgAdmin → Create Server → Connection tab:
# Host: 10.10.40.30
# Port: 5432
# Maintenance DB: postgres
# Username: webadmin
# Password: T@ylorSwift13
# SSL: Prefer/Disable
# Click Save

## 9. Verify Everything Works
# Check PostgreSQL listening:
ss -tlnp | grep 5432
# Check roles:
sudo -u postgres psql -c "\du"
# pgAdmin should now connect successfully

## 10. Summary
# - PostgreSQL listens on all interfaces
# - User 'webadmin' created with LOGIN + SUPERUSER
# - pg_hba.conf allows LAN + Docker subnets
# - Firewall allows port 5432
# - pgAdmin Docker container can now connect

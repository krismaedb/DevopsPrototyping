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

---
---

## 🍃 Flask + PostgreSQL Deployment (Ubuntu Server)  
**Project:** Healthcare Clinic Portal  
**Status:** WORKING (Flask + venv + PostgreSQL + Routes)  
**Format:** Single-file documentation

---

# 📌 1. Create Project Directory
```bash
sudo mkdir -p /var/www/healthclinic
sudo chown -R webadmin:webadmin /var/www/healthclinic
cd /var/www/healthclinic
```
Why:
We need a folder to keep all our Flask code, database connections, templates, and static files.
chown → makes you the owner so you can read/write without sudo.
cd → move into the project folder.

---

# 📌 2. Create Virtual Environment
```bash
python3 -m venv venv
source venv/bin/activate
pip install flask flask_sqlalchemy psycopg2-binary
```
Why:
Virtual environment isolates Python packages for this project → no conflict with other projects.
flask → the web framework.
flask_sqlalchemy → makes interacting with PostgreSQL easier.
psycopg2-binary → the driver that lets Python talk to PostgreSQL.

---

# 📌 3. PostgreSQL Setup

## Create PostgreSQL role and DB
```bash
sudo -u postgres psql
```

Inside psql:
```sql
CREATE ROLE webadmin WITH LOGIN SUPERUSER PASSWORD 'T@ylorSwift13';
CREATE DATABASE healthclinic OWNER webadmin;
\q
```
Why:
CREATE ROLE → make a PostgreSQL user with a password.
SUPERUSER → so it can create tables, etc.
CREATE DATABASE → make a database called healthclinic.
OWNER webadmin → this user owns the DB.

To verify:
```bash
sudo -u postgres psql -c "\du"
```

---

# 📌 4. Flask App Structure
Directory layout:

```
/var/www/healthclinic/
│ run.py
│
└── app/
    │ __init__.py
    │ models.py
    │ routes.py
    │
    ├── templates/
    └── static/
```
Why:
run.py → starts the Flask server.
__init__.py → sets up Flask + database connection.
models.py → defines tables like Patient, Employee, etc.
routes.py → defines web pages / API endpoints.
templates/ → HTML files.
static/ → CSS, JS, images.

---

# 📌 5. `app/__init__.py`
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

def create_app():
    app = Flask(__name__)

    # PostgreSQL connection (password encoded @ → %40)
    app.config["SQLALCHEMY_DATABASE_URI"] = (
        "postgresql://webadmin:T%40ylorSwift13@127.0.0.1:5432/healthclinic"
    )
    app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False

    db.init_app(app)

    # Register routes
    try:
        from .routes import main_bp
        app.register_blueprint(main_bp)
    except ImportError:
        pass

    return app
```
Why:
This tells Flask how to connect to PostgreSQL.
Notice the %40 → @ is URL encoded because passwords with @ need encoding.
db.init_app(app) → connects SQLAlchemy to Flask.

---

# 📌 6. `app/models.py`
```python
from . import db

class Patient(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(120), nullable=False)
    birthdate = db.Column(db.String(20))
```
Why:
db.Model → each class is a database table.
id → primary key (unique identifier).
name → a field for the patient’s name.
This is how Flask knows what tables to create in PostgreSQL.

---

# 📌 7. `app/routes.py`
```python
from flask import Blueprint

main_bp = Blueprint("main", __name__)

@main_bp.route("/")
def index():
    return "Healthcare Clinic Portal is running!"
```
Why:
This defines a web page route.
/ → root page.
When you go to http://IP:5000/ in browser, you see this message.

---

# 📌 8. Create DB Tables
Run Python shell INSIDE venv:

```bash
cd /var/www/healthclinic
source venv/bin/activate
python3
```

Inside Python:
```python
from app import create_app
from app.models import db

app = create_app()

with app.app_context():
    db.create_all()
```

Exit:
```python
quit()
```
Why:
This reads models.py and creates tables in PostgreSQL automatically.
app.app_context() → Flask needs this so SQLAlchemy knows which app it belongs to.

---

# 📌 9. `run.py` (Flask entry point)
```python
from app import create_app

app = create_app()

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)
```
Why:
0.0.0.0 → makes Flask accessible from outside, not just localhost.
port=5000 → the port the app listens on.
debug=True → shows errors in browser, auto-reloads on code change.

---

# 📌 10. Run Flask
```bash
cd /var/www/healthclinic
source venv/bin/activate
python3 run.py
```
Why:
Starts the Flask server so you can open your browser and see the app.

---

# 📌 11. Test  
Visit:

```
http://10.10.40.30:5000/
```

Expected:

```
Healthcare Clinic Portal is running!
```
Why:
Confirms your Flask app is running and connected to PostgreSQL correctly.

###  PART 8: Apache WSGI Configuration 

# Why We Added Apache + WSGI

In this section, we set up Apache and WSGI so our Flask system can run like a real website. Flask alone is good for development, but it is not designed to run a production website by itself. Apache + WSGI makes the application stable, secure, and always running in the background.

---

# 📌 1. Project Folder Structure

Your /var/www/healthclinic should look like:

/var/www/healthclinic
* app/
    * init.py
    * ... (other app files, e.g., routes.py, models.py)
* venv/
* **healthclinic.wsgi**
* requirements.txt

---
# Why We Need Apache

Apache is a web server.
It receives requests from the browser (ex. http://10.10.40.30) and delivers the correct response.

Flask can do this, but only for testing.
Apache is better because:
- It runs the website 24/7
- It can handle many users at the same time
- It can manage logs and errors
- It is the standard setup for real deployments

# 📌 2. Create the WSGI File

Location: /var/www/healthclinic/healthclinic.wsgi

import sys
import logging
logging.basicConfig(stream=sys.stderr)

# Insert the project directory into the Python path
sys.path.insert(0, "/var/www/healthclinic")

# Import the application factory function
from app import create_app

# The 'application' variable name is required by mod_wsgi
application = create_app()

**✔ Explanation**
sys.path.insert(0, "...") → tells Apache where your Flask project folder is.
create_app() → loads your Flask factory application.
application → required variable name for mod_wsgi.

---

## 📌 3. Apache Virtual Host Config

Location: /etc/apache2/sites-available/healthclinic.conf

<VirtualHost *:80>
    ServerName 10.10.40.30

    # Define the daemon process group for the application
    WSGIDaemonProcess healthclinic \
        python-home=/var/www/healthclinic/venv \
        python-path=/var/www/healthclinic

    # Assign the WSGI script to the defined daemon process group
    WSGIProcessGroup healthclinic
    # Map the root URL (/) to the WSGI script
    WSGIScriptAlias / /var/www/healthclinic/healthclinic.wsgi

    # Allow access to the WSGI script directory
    <Directory /var/www/healthclinic>
        Require all granted
    </Directory>

    # Log files
    ErrorLog ${APACHE_LOG_DIR}/healthclinic_error.log
    CustomLog ${APACHE_LOG_DIR}/healthclinic_access.log combined
</VirtualHost>

**✔ Explanation**
WSGIDaemonProcess: Creates a dedicated process for your app.
python-home: Path to your application's **virtual environment** (venv).
python-path: Path to your Flask **project folder**.
WSGIScriptAlias: Tells Apache which **WSGI file** to load and the URL path to serve it from (/).
<Directory>: Grants access to the file path.
Logs: Essential for debugging errors.

---

## 📌 4. Enable the Site + Restart Apache

Run the following commands:

# Ensure the mod_wsgi module is enabled
sudo a2enmod wsgi

# Enable the virtual host configuration file
sudo a2ensite healthclinic.conf

# Restart the Apache web server to apply changes
sudo systemctl restart apache2

---

## 📌 5. Support Commands for Debugging

Check config syntax:
sudo apache2ctl configtest

View error logs:
sudo tail -n 50 /var/log/apache2/healthclinic_error.log

---

## 📌 6. Final Result

If everything is correct, visiting: [http://10.10.40.30](http://10.10.40.30)

will show: Healthcare Clinic Portal is running!

successfully deployed using Apache + WSGI!


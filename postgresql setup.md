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

Why We Need WSGI
- WSGI is the connector between Apache and Flask.
- Apache doesn’t understand Python by default.
- WSGI acts like a “translator” that tells Apache how to run our Python Flask application.

So the flow becomes:
Browser → Apache → WSGI → Flask → Database

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

Why We Created healthclinic.wsgi
This file is the entry point for Apache.
It tells Apache:
- where the Flask project is located
- how to load the create_app() function
- what variable to run as the application
- Apache reads this file and launches the Flask system automatically.

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
Why We Made a Virtual Host (healthclinic.conf)
The Apache VirtualHost configuration defines:
- the server IP or domain
- the folder where the project is located
- the virtual environment to use
- the WSGI file that starts the app
- the access permissions

In short:
It tells Apache “this is where the website is, and this is how you run it.”

## 📌 4. Enable the Site + Restart Apache

Run the following commands:

# Ensure the mod_wsgi module is enabled
sudo a2enmod wsgi

# Enable the virtual host configuration file
sudo a2ensite healthclinic.conf

# Restart the Apache web server to apply changes
sudo systemctl restart apache2

---

Why We Enabled the Site and Restarted Apache
After creating the configuration file, we must enable it so Apache recognizes it.
Restarting Apache reloads all the new settings.
Once enabled, the Flask application becomes accessible through:

http://10.10.40.30

(Port 80 — the standard web port)
This is why the application no longer runs on port 5000.

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

** The server will display the full Healthcare Clinic web interface once Your current deployed version = full GUI landing page

# Final Summary (For Documentation)
Apache + WSGI allows our Flask + PostgreSQL system to run like a production website. Flask is still the core application, but Apache handles all incoming web requests and uses WSGI to communicate with the Python backend. This setup ensures stability, proper resource handling, and accessibility through a normal web browser.

📘 Phase 1 – Landing Page Setup (SOP Documentation)

Standard Operating Procedure — HealthClinic Flask Web UI

# Phase 1 — Landing Page Setup

This document describes the steps to build the public Landing Page for the HealthClinic Flask Application.



----

## HIGH‑LEVEL DIAGRAM: How Everything Connects

             ┌──────────────────────────┐
             │        Browser           │
             │ (User visits the site)   │
             └──────────────┬───────────┘
                            │  http://10.10.40.30
                            ▼
             ┌──────────────────────────┐
             │         Apache           │
             │    (Production Webserver)│
             └──────────────┬───────────┘
                            │  passes request into
                            │
                            ▼
             ┌──────────────────────────┐
             │           WSGI           │
             │ (Apache ↔ Flask bridge)  │
             └──────────────┬───────────┘
                            │  runs your Flask app
                            ▼
             ┌──────────────────────────┐
             │           Flask          │
             │    (Python backend)      │
             └──────────────┬───────────┘
                            │ uses SQLAlchemy ORM
                            ▼
             ┌──────────────────────────┐
             │        PostgreSQL        │
             │ (Database for clinic)    │
             └──────────────┬───────────┘
                            │
                       (optional)
                            ▼
             ┌──────────────────────────┐
             │          pgAdmin         │
             │  (DB GUI via Docker)     │
             └──────────────────────────┘
Everything above works together like this:

Apache → WSGI → Flask → SQLAlchemy → PostgreSQL
pgAdmin is only for manual database management.
ORM converts your Python code → into SQL → runs it → and gives the result back as a Python object. 

# 1. PostgreSQL + pgAdmin Section (Docker)
Purpose:
Set up database server (PostgreSQL)
Allow remote tools (pgAdmin) to connect
Create DB user + db for Flask
Flask needs a database to store: patients, appointments, users, logs, etc. -> PostgreSQL is that database.
Create a PostgreSQL user -> Gumawa tayo ng account sa PostgreSQL para si Flask makapasok.
Create the PostgreSQL database where Flask will store its tables -> Ito yung real database kung saan ilalagay ang tables (Patients, Appointment, Users).

How it connects:
✔ PostgreSQL stores all patient, appointment, user, and logs
✔ Flask reads/writes data to PostgreSQL
✔ pgAdmin connects to PostgreSQL for GUI access
✔ Docker pgAdmin is a separate container but talks to PostgreSQL via port 5432 - because they are same database but NOT the same container
🧩 PostgreSQL = database server
Running directly on Ubuntu (NOT in Docker)

🧩 pgAdmin = GUI tool
Running inside Docker
🚫 It is NOT the database
✔ It is only a viewer (like PhpMyAdmin)

They are SEPARATE, but:
pgAdmin → connects via → 10.10.40.30:5432 → PostgreSQL

Exactly like this diagram:
[pgAdmin container]  --5432-->  [PostgreSQL on Ubuntu]
So yes, they share the same DB, but pgAdmin is only a “remote control panel”.


# 2. Flask + Virtual Environment Section
Purpose:
Install Flask and Python dependencies
Keep packages isolated
Run your backend locally (before Apache)

How it connects:
✔ Flask connects to PostgreSQL using SQLAlchemy
✔ Flask runs inside the project directory /var/www/healthclinic
✔ When deployed, Flask is run by WSGI, not by python run.py


# 3. App Folder Structure
Purpose:
Organizes code:
- __init__.py → creates Flask app + DB connection
- routes.py → defines website URLs
- models.py → defines database tables
- templates/ → HTML files
- static/ → images, CSS, JS

How it connects:
✔ Apache loads Flask via WSGI → which loads create_app() from here
✔ SQLAlchemy uses models.py to create PostgreSQL tables


# 4. SQLAlchemy + PostgreSQL Connection (init.py)
Purpose:
Connect Flask to PostgreSQL
Initialize SQLAlchemy instance
Store DB URI
SQLAlchemy “DB URI” - Uniform Resource Identifier - “how Flask connects to the database”
Your URI:
postgresql://webadmin:T%40ylorSwift13@127.0.0.1:5432/healthclinic
*** This string is stored in __init__.py ***

How it connects:
✔ SQLAlchemy handles all INSERT, SELECT, UPDATE, DELETE
✔ PostgreSQL receives these operations
✔ Flask functions in routes.py call SQLAlchemy
This does:
Flask receives form data
SQLAlchemy turns Python objects → SQL commands
SQLAlchemy inserts into PostgreSQL
PostgreSQL stores data

Flow:
User submits form → Flask route → SQLAlchemy → PostgreSQL database

# 5. Creating Tables with db.create_all()
Purpose:
Build database tables automatically using models.

How it connects:
✔ Flask app context → required so SQLAlchemy knows which app is running
✔ PostgreSQL receives commands to create tables based on models

# 6. run.py (Development mode)
Purpose:
Run Flask manually with:

python3 run.py
How it connects:
✔ Used only for local testing before deployment
❗ NOT used after Apache is installed

Apache will replace it.
“So we don’t use python run.py anymore?” - Correct.
Now:
✔ Apache runs Flask
✔ healthclinic.wsgi is the Flask entry
✔ run.py is deprecated (pang-testing lang)
You can delete run.py if you want; it’s not used.

# 7. Apache + WSGI (Production mode)
This is where most people get confused — but here's the simple version:

🔥 Apache (Web Server)
Purpose:
Accepts web traffic on port 80
Routes requests to WSGI
Serves HTML pages and static files

How it connects:
✔ When user visits http://10.10.40.30, Apache receives it
✔ Apache passes the request to WSGI

🔥 WSGI (Connector)
Purpose:
Connects Apache ↔ Flask
Loads your app usingvcreate_app()
WSGI file example:

application = create_app()
How it connects:
✔ Apache executes the WSGI script
✔ WSGI runs the Flask app
✔ Flask queries PostgreSQL
✔ Response returns back to the browser

# 8. Apache VirtualHost Configuration - This is the part where you deployed Flask into a real web server
Steps:
1. Install Apache
2. Install mod_wsgi
3. Create config:
/etc/apache2/sites-available/healthclinic.conf
4. Enable site:
5. sudo a2ensite healthclinic.conf
6. Restart Apache
7. Done → system is now LIVE

Purpose:
Tell Apache:
Where the Flask code is
Where the virtual environment is
What WSGI file to run
How to serve the website

How it connects:
✔ Ties everything together
✔ Without this file, Apache cannot run Flask
✔ After enabling it, the site runs automatically

# 9. Final Deployment Flow
Here’s the final working flow:
User → Apache → WSGI → Flask app → SQLAlchemy → PostgreSQL

And optionally:
pgAdmin → PostgreSQL (for admin GUI)

# 🎯 THE MOST IMPORTANT PARTS TO HIGHLIGHT IN SOP
If you're documenting and want it CLEAN:

A. Flask ↔ PostgreSQL
- SQLAlchemy handles the connection
- DB URI is defined inside __init__.py
- PostgreSQL stores all data

B. Apache ↔ WSGI ↔ Flask
- Apache listens on port 80
- WSGI loads the Flask app
- Flask processes routes and talks to PostgreSQL

C. pgAdmin
- Not part of the website
- Only for DB administration
- Runs via Docker container

D. Folder Structure
/var/www/healthclinic
venv for python
app/ for Flask code
healthclinic.wsgi for Apache















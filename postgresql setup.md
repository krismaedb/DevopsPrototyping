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

# 🍃 Flask + PostgreSQL Deployment (Ubuntu Server)  
**Project:** Healthcare Clinic Portal  
**Status:** WORKING (Flask + venv + PostgreSQL + Routes)  
**Format:** Single-file documentation

---

## 📌 1. Create Project Directory
```bash
sudo mkdir -p /var/www/healthclinic
sudo chown -R webadmin:webadmin /var/www/healthclinic
cd /var/www/healthclinic
```

---

## 📌 2. Create Virtual Environment
```bash
python3 -m venv venv
source venv/bin/activate
pip install flask flask_sqlalchemy psycopg2-binary
```

---

## 📌 3. PostgreSQL Setup

### Create PostgreSQL role and DB
```bash
sudo -u postgres psql
```

Inside psql:
```sql
CREATE ROLE webadmin WITH LOGIN SUPERUSER PASSWORD 'T@ylorSwift13';
CREATE DATABASE healthclinic OWNER webadmin;
\q
```

To verify:
```bash
sudo -u postgres psql -c "\du"
```

---

## 📌 4. Flask App Structure
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

---

## 📌 5. `app/__init__.py`
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

---

## 📌 6. `app/models.py`
```python
from . import db

class Patient(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(120), nullable=False)
    birthdate = db.Column(db.String(20))
```

---

## 📌 7. `app/routes.py`
```python
from flask import Blueprint

main_bp = Blueprint("main", __name__)

@main_bp.route("/")
def index():
    return "Healthcare Clinic Portal is running!"
```

---

## 📌 8. Create DB Tables
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

---

## 📌 9. `run.py` (Flask entry point)
```python
from app import create_app

app = create_app()

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)
```

---

## 📌 10. Run Flask
```bash
cd /var/www/healthclinic
source venv/bin/activate
python3 run.py
```

---

## 📌 11. Test  
Visit:

```
http://YOUR-SERVER-IP:5000/
```

Expected:

```
Healthcare Clinic Portal is running!
```

---

## ✔️ Deployment SUCCESS  
This `.md` contains everything we did:  
setup → venv → install → DB → routes → run.



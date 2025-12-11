# 📘 WEB01 Healthcare Clinic - Technical Documentation

## 📋 Table of Contents

1. [System Overview](#system-overview)
2. [Architecture & Technology Stack](#architecture--technology-stack)
3. [Installation Guide](#installation-guide)
4. [Database Schema](#database-schema)
5. [Features & User Guide](#features--user-guide)
6. [Administration](#administration)
7. [Troubleshooting](#troubleshooting)

---

## System Overview

### Project Description
WEB01 is a web application server hosting a patient portal and appointment management system for Healthcare Clinic.

### Key Features
- ✅ Patient registration and management (CRUD operations)
- ✅ Online appointment booking and scheduling
- ✅ Role-based authentication (Admin, IT, Doctor, Nurse)
- ✅ Staff dashboard with real-time statistics
- ✅ Responsive Bootstrap 5 interface
- ✅ PostgreSQL database with relational data model

### Server Specifications

| Component | Specification |
|-----------|--------------|
| **VM Platform** | Proxmox VE (VM ID: 130) |
| **CPU** | 4 cores |
| **RAM** | 8 GB |
| **Storage** | 80 GB |
| **Network** | 10.10.40.30/24 (VLAN 40) |
| **OS** | Ubuntu 24.04 LTS Server |
| **Hostname** | web01.healthclinic.local |

---

## Architecture & Technology Stack

### System Architecture
```
┌─────────────────┐
│   Web Browser   │
└────────┬────────┘
         │ HTTP (Port 80)
         ▼
┌─────────────────┐
│  Apache 2.4     │ ← Web Server
└────────┬────────┘
         │ mod_wsgi
         ▼
┌─────────────────┐
│  Flask 3.0      │ ← Python Web Framework
│  - Routes       │
│  - Models       │
│  - Templates    │
└────────┬────────┘
         │ SQLAlchemy ORM
         ▼
┌─────────────────┐
│ PostgreSQL 15   │ ← Database
│ - users         │
│ - patients      │
│ - appointments  │
└─────────────────┘
```

### Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| **Frontend** | Bootstrap | 5.3.2 |
| **Frontend** | Font Awesome | 6.5.1 |
| **Backend** | Flask | 3.0.0 |
| **ORM** | SQLAlchemy | 2.0.23 |
| **Auth** | Flask-Login | 0.6.3 |
| **Database** | PostgreSQL | 15.5 |
| **Web Server** | Apache | 2.4.58 |

### Network Configuration
```
IP Address:     10.10.40.30/24
Gateway:        10.10.40.1
DNS Primary:    10.10.40.10
DNS Secondary:  10.10.40.11
Domain:         healthclinic.local
```

---

## Installation Guide

### Prerequisites
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install required packages
sudo apt install -y apache2 postgresql postgresql-contrib python3 python3-pip python3-venv \
                    libapache2-mod-wsgi-py3 build-essential python3-dev
```

### PostgreSQL Setup
```bash
# Create database and user
sudo -u postgres psql << 'EOF'
CREATE ROLE webadmin WITH LOGIN PASSWORD 'T@ylorSwift13';
ALTER ROLE webadmin WITH SUPERUSER;
CREATE DATABASE healthclinic OWNER webadmin;
\c healthclinic
GRANT ALL ON SCHEMA public TO webadmin;
\q
EOF
```

### Flask Application Setup
```bash
# Create project directory
sudo mkdir -p /var/www/healthclinic
sudo chown -R webadmin:www-data /var/www/healthclinic
cd /var/www/healthclinic

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install Python packages
pip install Flask Flask-SQLAlchemy Flask-Login psycopg2-binary Werkzeug
```

### Apache Configuration
```bash
# Create WSGI file
cat > /var/www/healthclinic/healthclinic.wsgi << 'EOF'
import sys
sys.path.insert(0, "/var/www/healthclinic")
from app import create_app
application = create_app()
EOF

# Create Apache virtual host
sudo bash -c 'cat > /etc/apache2/sites-available/healthclinic.conf << "APACHEEOF"
<VirtualHost *:80>
    ServerName web01.healthclinic.local
    ServerAlias 10.10.40.30

    WSGIDaemonProcess healthclinic python-home=/var/www/healthclinic/venv python-path=/var/www/healthclinic
    WSGIProcessGroup healthclinic
    WSGIScriptAlias / /var/www/healthclinic/healthclinic.wsgi

    <Directory /var/www/healthclinic>
        Require all granted
    </Directory>

    Alias /static /var/www/healthclinic/app/static
    <Directory /var/www/healthclinic/app/static>
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/healthclinic_error.log
    CustomLog ${APACHE_LOG_DIR}/healthclinic_access.log combined
</VirtualHost>
APACHEEOF'

# Enable site and restart Apache
sudo a2dissite 000-default.conf
sudo a2ensite healthclinic.conf
sudo a2enmod wsgi
sudo systemctl restart apache2
```

### Initialize Database
```bash
cd /var/www/healthclinic
source venv/bin/activate

# Create tables
python3 << 'EOF'
from app import create_app, db
app = create_app()
with app.app_context():
    db.create_all()
    print("✅ Database tables created!")
EOF

# Create default users
python3 app/create_user.py
```

---

## Database Schema

### Entity Relationship Diagram
```
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│     USERS       │       │    PATIENTS     │       │  APPOINTMENTS   │
├─────────────────┤       ├─────────────────┤       ├─────────────────┤
│ PK id           │       │ PK id           │───┐   │ PK id           │
│ UQ username     │       │ UQ patient_id   │   └───│ FK patient_id   │
│ UQ email        │       │    first_name   │       │    patient_name │
│    password_hash│       │    last_name    │       │    appt_date    │
│    full_name    │       │    dob          │       │    appt_time    │
│    role         │       │    gender       │       │    doctor       │
│    phone        │       │    phone        │       │    status       │
│    is_active    │       │    email        │       │    reason       │
│    created_at   │       │    address      │       │    created_at   │
└─────────────────┘       └─────────────────┘       └─────────────────┘
```

### Table Definitions

**Users Table:**
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(120) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(100) NOT NULL,
    role VARCHAR(20) DEFAULT 'nurse',
    phone VARCHAR(20),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP
);
```

**Patients Table:**
```sql
CREATE TABLE patients (
    id SERIAL PRIMARY KEY,
    patient_id VARCHAR(20) UNIQUE NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    date_of_birth DATE,
    gender VARCHAR(10),
    phone VARCHAR(20),
    email VARCHAR(120),
    address TEXT,
    emergency_contact VARCHAR(100),
    emergency_phone VARCHAR(20),
    blood_type VARCHAR(5),
    allergies TEXT,
    medical_notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Appointments Table:**
```sql
CREATE TABLE appointments (
    id SERIAL PRIMARY KEY,
    patient_id INTEGER REFERENCES patients(id) ON DELETE CASCADE,
    patient_name VARCHAR(100) NOT NULL,
    patient_email VARCHAR(120),
    patient_phone VARCHAR(20),
    appointment_date DATE NOT NULL,
    appointment_time VARCHAR(10) NOT NULL,
    doctor VARCHAR(50) NOT NULL,
    department VARCHAR(50),
    reason TEXT NOT NULL,
    status VARCHAR(20) DEFAULT 'pending',
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Features & User Guide

### Public Features

**Landing Page** (`http://10.10.40.30`)
- Welcome message with clinic information
- Service overview cards
- Statistics display
- Call-to-action buttons

**Appointment Booking** (`/appointment`)
- Patient information form (name, email, phone)
- Date and time selection
- Doctor selection
- Reason for visit
- No login required

### Staff Features (Login Required)

**Dashboard** (`/dashboard`)
- Statistics cards:
  - Total Patients
  - Today's Appointments
  - Pending Appointments
  - Total Appointments
- Recent appointments table
- Quick action buttons

**Patient Management** (`/patients`)
- List all patients with search
- Add new patient (auto-generates ID: P00001, P00002...)
- View patient details
- Edit patient information
- Delete patient (with confirmation)

**Appointment Management** (`/appointments`)
- List all appointments
- Filter by status and date
- Book appointment (linked to patients)
- View appointment details
- Update appointment status
- Delete appointment

### User Roles

| Role | Access Level |
|------|-------------|
| **Admin** | Full access to all features |
| **IT** | Full access + system monitoring |
| **Doctor** | Patients, appointments, own schedule |
| **Nurse** | Patients, appointments |

### Default User Accounts
```
Username: admin       Password: g3company!@#    Role: Administrator
Username: psantos     Password: g3company!@#    Role: IT Staff
Username: dr.johnson  Password: g3company!@#    Role: Doctor
Username: nurse.maria Password: g3company!@#    Role: Nurse
```

---

## Administration

### Service Management
```bash
# Apache
sudo systemctl start apache2
sudo systemctl stop apache2
sudo systemctl restart apache2
sudo systemctl status apache2

# PostgreSQL
sudo systemctl start postgresql
sudo systemctl stop postgresql
sudo systemctl restart postgresql
sudo systemctl status postgresql
```

### Database Operations
```bash
# Connect to database
psql -h localhost -U webadmin -d healthclinic
# Password: T@ylorSwift13

# Common SQL commands
\dt                          # List tables
\d users                     # Describe users table
SELECT * FROM users;         # View all users
SELECT COUNT(*) FROM patients; # Count patients
\q                           # Quit
```

### User Management
```bash
# List all users
cd /var/www/healthclinic
source venv/bin/activate
python3 app/create_user.py --list

# Reset all passwords
python3 app/create_user.py --reset-passwords
```

### Backup Database
```bash
# Create backup
pg_dump -h localhost -U webadmin healthclinic > backup_$(date +%Y%m%d).sql

# Restore backup
psql -h localhost -U webadmin -d healthclinic < backup_20241209.sql
```

### View Logs
```bash
# Apache error log
sudo tail -f /var/log/apache2/healthclinic_error.log

# Apache access log
sudo tail -f /var/log/apache2/healthclinic_access.log
```

---

## Troubleshooting

### Website Not Accessible
```bash
# Check Apache status
sudo systemctl status apache2

# Restart Apache
sudo systemctl restart apache2

# Check if port 80 is listening
sudo ss -tulpn | grep :80

# Test configuration
sudo apache2ctl configtest
```

### Internal Server Error (500)
```bash
# View error log
sudo tail -100 /var/log/apache2/healthclinic_error.log

# Fix permissions
sudo chown -R webadmin:www-data /var/www/healthclinic
sudo chmod -R 755 /var/www/healthclinic

# Restart Apache
sudo systemctl restart apache2
```

### Database Connection Error
```bash
# Check PostgreSQL status
sudo systemctl status postgresql

# Test connection
psql -h localhost -U webadmin -d healthclinic -c "SELECT 1;"

# Restart PostgreSQL
sudo systemctl restart postgresql
```

### Login Not Working
```bash
# Check if users exist
psql -h localhost -U webadmin -d healthclinic -c "SELECT username, role FROM users;"

# Reset user password
cd /var/www/healthclinic
source venv/bin/activate
python3 << 'EOF'
from app import create_app, db
from app.models import User
app = create_app()
with app.app_context():
    user = User.query.filter_by(username='admin').first()
    user.set_password('g3company!@#')
    db.session.commit()
    print('Password reset successfully')
EOF
```

### CSS Not Loading
```bash
# Check static files
ls -la /var/www/healthclinic/app/static/

# Fix permissions
sudo chown -R webadmin:www-data /var/www/healthclinic/app/static
sudo chmod -R 755 /var/www/healthclinic/app/static

# Clear browser cache (Ctrl+Shift+R)
```

---

## Quick Reference

### URLs
```
Public Site:     http://10.10.40.30
Staff Login:     http://10.10.40.30/login
Dashboard:       http://10.10.40.30/dashboard
Patients:        http://10.10.40.30/patients
Appointments:    http://10.10.40.30/appointments
```

### File Locations
```
Application:     /var/www/healthclinic/
Virtual Env:     /var/www/healthclinic/venv/
Templates:       /var/www/healthclinic/app/templates/
Static Files:    /var/www/healthclinic/app/static/
WSGI File:       /var/www/healthclinic/healthclinic.wsgi
Apache Config:   /etc/apache2/sites-available/healthclinic.conf
Error Log:       /var/log/apache2/healthclinic_error.log
```

### Project Structure
```
/var/www/healthclinic/
├── app/
│   ├── __init__.py          # Flask app factory
│   ├── models.py            # Database models
│   ├── routes.py            # URL routes
│   ├── create_user.py       # User management script
│   ├── templates/           # HTML templates
│   │   ├── base_dashboard.html
│   │   ├── dashboard.html
│   │   ├── index.html
│   │   ├── login.html
│   │   ├── patients_list.html
│   │   ├── patients_add.html
│   │   ├── patients_view.html
│   │   ├── patients_edit.html
│   │   ├── appointments_list.html
│   │   ├── appointments_book.html
│   │   └── appointments_view.html
│   └── static/
│       └── css/
│           └── style.css
├── venv/                    # Python virtual environment
└── healthclinic.wsgi       # WSGI entry point
```

---

## Security Implementation

### Password Security
- Werkzeug password hashing (pbkdf2:sha256)
- Salted hashes stored in database
- No plaintext passwords

### Authentication
- Flask-Login session management
- Login required decorators on protected routes
- Role-based access control

### Database Security
- PostgreSQL password authentication
- Limited network access (localhost only from Flask)
- Parameterized queries (SQLAlchemy ORM prevents SQL injection)

### Network Security
- Firewall rules: Port 80 (HTTP) open, Port 22 (SSH) VLAN 40 only
- PostgreSQL port 5432 localhost only


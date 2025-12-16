# PostgreSQL Database Documentation

**Project:** Healthcare Clinic Capstone  
**Institution:** Manitoba Institute of Trades and Technology (M.I.T.T.)  
**Program:** Network and Systems Administrator Diploma  
**Team:** Kristine Bagsican, Swathi Anil, Kunwei Li, Jaered Bacolod
**Team Size:** 4 members  
**Duration:** 2 weeks  
**Location:** Winnipeg, Manitoba, Canada

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Network Architecture](#network-architecture)
3. [Physical Infrastructure](#physical-infrastructure)
4. [Network Devices Configuration](#network-devices-configuration)
5. [Server Infrastructure](#server-infrastructure)
6. [Database Setup & Management](#database-setup--management)
7. [Security Implementation](#security-implementation)
8. [Monitoring & Management](#monitoring--management)
9. [Backup & Disaster Recovery](#backup--disaster-recovery)
10. [Testing & Validation](#testing--validation)
11. [Troubleshooting Guide](#troubleshooting-guide)
12. [Appendices](#appendices)

---

## 1. Executive Summary

### 1.1 Project Overview

The Healthcare Clinic IT Infrastructure project delivers a complete, production-ready network environment supporting:

- **200+ users** across 4 VLANs (Patient WiFi, Clinical Staff, Administration, IT/Servers)
- **High availability** through HSRP, EtherChannel, and redundant domain controllers
- **Secure segmentation** with ACLs and firewall policies
- **Centralized monitoring** via Zabbix
- **Email services** through Mailcow on Debian
- **Web applications** using Flask/PostgreSQL
- **File storage** with TrueNAS SCALE
- **Remote access** via Tailscale VPN

### 1.2 Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Network** | Cisco ISR 2901/4321, Catalyst 2960 | Routing, switching, VLAN segmentation |
| **Virtualization** | Proxmox VE | Host VMs and containers |
| **Directory Services** | Active Directory (Windows Server 2022) | Authentication, GPO, DNS, DHCP |
| **Monitoring** | Zabbix 7.4 | Infrastructure monitoring |
| **Email** | Mailcow (Docker) | Email server with webmail |
| **Web Server** | Apache + Flask + PostgreSQL | Patient portal |
| **Storage** | TrueNAS SCALE | Network-attached storage |
| **VPN** | Tailscale | Secure remote access |
| **Database** | PostgreSQL 15 | Relational database for web applications |

### 1.3 Key Achievements

  **99.9% uptime** through redundant routers and domain controllers  
  **Zero-trust segmentation** with VLAN isolation and ACLs  
  **Automated monitoring** of 9 devices (4 Cisco, 2 Windows, 3 Linux)  
  **Secure remote access** without exposing public IPs  
  **Centralized authentication** with Active Directory  
  **Professional email system** with spam protection  
  **Web-based patient portal** with appointment booking  
  **20TB storage capacity** with ZFS redundancy  
  **Production-grade databases** with PostgreSQL for web and monitoring systems

---

## 6. Database Setup & Management

### 6.1 Database Architecture Overview
```
┌─────────────────────────────────────────────────────────────┐
│                  DATABASE ARCHITECTURE                       │
└─────────────────────────────────────────────────────────────┘

                    ┌──────────────────┐
                    │   Web Browsers   │
                    │  (HTTP Clients)  │
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
                    │   Apache Server  │
                    │   (Port 80)      │
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
                    │   WSGI Bridge    │
                    │ (mod_wsgi_py3)   │
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
                    │  Flask App       │
                    │  (Python)        │
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
                    │  SQLAlchemy ORM  │
                    │  (Database API)  │
                    └────────┬─────────┘
                             │
        ┌────────────────────┴────────────────────┐
        │                                         │
┌───────▼───────┐                       ┌─────────▼────────┐
│  PostgreSQL   │                       │   PostgreSQL     │
│   (WEB01)     │                       │   (ZABBIX01)     │
│               │                       │                  │
│ healthclinic  │                       │    zabbix        │
│   database    │                       │   database       │
│               │                       │                  │
│ - users       │                       │ - hosts          │
│ - patients    │                       │ - items          │
│ - appointments│                       │ - triggers       │
│ - audit_logs  │                       │ - events         │
└───────┬───────┘                       └─────────┬────────┘
        │                                         │
        │         ┌──────────────────┐           │
        └─────────►   pgAdmin GUI    ◄───────────┘
                  │  (Docker)        │
                  │  Port: 5050      │
                  └──────────────────┘
```

### 6.2 PostgreSQL Installation & Configuration

#### 6.2.1 WEB01 - PostgreSQL Setup

**Purpose:** Database for Healthcare Clinic Patient Portal

**Install PostgreSQL 15:**
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install PostgreSQL
sudo apt install -y postgresql postgresql-contrib

# Verify installation
psql --version
# Expected: psql (PostgreSQL) 15.x

# Check service status
sudo systemctl status postgresql
```

**Configure PostgreSQL for Remote Access:**
```bash
# Edit postgresql.conf to listen on all interfaces
sudo nano /etc/postgresql/15/main/postgresql.conf

# Find and modify:
listen_addresses = '*'          # Changed from 'localhost'
max_connections = 100           # Maximum concurrent connections
shared_buffers = 256MB          # Memory for caching
effective_cache_size = 1GB      # Estimated available cache
work_mem = 4MB                  # Memory per query operation
maintenance_work_mem = 64MB     # Memory for maintenance operations

# Save and exit (Ctrl+X, Y, Enter)
```

**Configure Authentication (pg_hba.conf):**
```bash
sudo nano /etc/postgresql/15/main/pg_hba.conf

# Add at the bottom (before the end):
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# Local connections
local   all             all                                     peer

# IPv4 local connections
host    all             all             127.0.0.1/32            md5

# Allow connections from VLAN 40 (Server network)
host    all             all             10.10.40.0/24           md5

# Allow connections from Docker bridge network (pgAdmin)
host    all             all             172.17.0.0/16           md5

# Optional: Allow all networks (DEVELOPMENT ONLY - NOT FOR PRODUCTION)
# host    all             all             0.0.0.0/0               md5

# Save and exit
```

**Restart PostgreSQL:**
```bash
sudo systemctl restart postgresql

# Verify listening on all interfaces
sudo ss -tlnp | grep 5432
# Expected output: 0.0.0.0:5432
```

**Create Database User and Database:**
```bash
# Switch to postgres user
sudo -u postgres psql

# Inside PostgreSQL prompt:
```
```sql
-- Create superuser for web application
CREATE ROLE webadmin WITH LOGIN SUPERUSER PASSWORD 'T@ylorSwift13';

-- Create database owned by webadmin
CREATE DATABASE healthclinic OWNER webadmin;

-- Connect to the database
\c healthclinic

-- Grant all privileges on schema
GRANT ALL PRIVILEGES ON SCHEMA public TO webadmin;

-- Grant all on existing tables
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO webadmin;

-- Grant all on sequences (for auto-increment)
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO webadmin;

-- Set default privileges for future objects
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO webadmin;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON SEQUENCES TO webadmin;

-- Verify user creation
\du

-- Expected output:
--                                   List of roles
--  Role name |                         Attributes                         
-- -----------+------------------------------------------------------------
--  postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS
--  webadmin  | Superuser

-- List databases
\l

-- Expected to see: healthclinic | webadmin

-- Exit PostgreSQL
\q
```

**Test Database Connection:**
```bash
# Test local connection
psql -h localhost -U webadmin -d healthclinic -W
# Enter password: T@ylorSwift13

# Inside psql prompt:
SELECT current_database();
SELECT current_user;

# Should output:
#  current_database 
# ------------------
#  healthclinic
# 
#  current_user 
# --------------
#  webadmin

\q

# Test remote connection from another machine
psql -h 10.10.40.30 -U webadmin -d healthclinic -W
# Should connect successfully
```

#### 6.2.2 ZABBIX01 - PostgreSQL Setup

**Purpose:** Database for Zabbix Monitoring System

**Install PostgreSQL 14:**
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install PostgreSQL 14
sudo apt install -y postgresql postgresql-contrib

# Verify installation
psql --version
# Expected: psql (PostgreSQL) 14.x

# Start and enable service
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo systemctl status postgresql
```

**Create Zabbix Database:**
```bash
# Switch to postgres user
sudo -u postgres psql
```
```sql
-- Drop existing database if any
DROP DATABASE IF EXISTS zabbix;

-- Create database with specific encoding
CREATE DATABASE zabbix 
    ENCODING 'UTF8' 
    LC_COLLATE 'en_US.UTF-8' 
    LC_CTYPE 'en_US.UTF-8' 
    TEMPLATE template0;

-- Create user
CREATE USER zabbixuser WITH PASSWORD 'g3company!@#';

-- Set database owner
ALTER DATABASE zabbix OWNER TO zabbixuser;

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE zabbix TO zabbixuser;

-- Connect to zabbix database
\c zabbix

-- Grant schema privileges
GRANT ALL ON SCHEMA public TO zabbixuser;

-- Verify
\l zabbix

-- Expected output:
--     Name    |   Owner    | Encoding |   Collate   |    Ctype    
-- ------------+------------+----------+-------------+-------------
--  zabbix     | zabbixuser | UTF8     | en_US.UTF-8 | en_US.UTF-8

\q
```

**Import Zabbix Database Schema:**
```bash
# Download Zabbix source for SQL files
cd /tmp
wget https://cdn.zabbix.com/zabbix/sources/stable/7.4/zabbix-7.4.5.tar.gz
tar -xzf zabbix-7.4.5.tar.gz
cd zabbix-7.4.5/database/postgresql/

# Import schema (creates table structure)
psql -h localhost -U zabbixuser -d zabbix -W < schema.sql
# Password: g3company!@#
# Wait 2-3 minutes

# Import images (icons, maps)
psql -h localhost -U zabbixuser -d zabbix -W < images.sql
# Password: g3company!@#

# Import initial data (templates, default configs)
psql -h localhost -U zabbixuser -d zabbix -W < data.sql
# Password: g3company!@#
```

**Grant Additional Permissions:**
```bash
sudo -u postgres psql -d zabbix
```
```sql
-- Grant all privileges on tables
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO zabbixuser;

-- Grant privileges on sequences
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO zabbixuser;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO zabbixuser;

-- Set default privileges for future objects
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO zabbixuser;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON SEQUENCES TO zabbixuser;

\q
```

**Verify Database Tables:**
```bash
# Count tables (should be 207)
sudo -u postgres psql -d zabbix -c "SELECT COUNT(*) FROM information_schema.tables WHERE table_schema = 'public';"

# Check database version
sudo -u postgres psql -d zabbix -c "SELECT * FROM public.dbversion;"

# Expected output:
#  dbversion | mandatory | optional 
# -----------+-----------+----------
#    7040000 |     7040000 |  7040000
```

**Configure Authentication:**
```bash
# Edit pg_hba.conf
sudo nano /etc/postgresql/14/main/pg_hba.conf

# Change peer and md5 authentication:
# local   all             all                                     md5
# host    all             all             127.0.0.1/32            md5
# host    all             all             ::1/128                 md5

# Restart PostgreSQL
sudo systemctl restart postgresql
```

**Test Zabbix Database Connection:**
```bash
# Test connection
psql -h localhost -U zabbixuser -d zabbix -W
# Password: g3company!@#

# Verify tables
\dt

# Should list 207 tables including:
# - hosts
# - items
# - triggers
# - events
# - history

\q
```

### 6.3 Database Design - Healthcare Clinic Portal

#### 6.3.1 Entity Relationship Diagram
```
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│     USERS       │       │    PATIENTS     │       │  APPOINTMENTS   │
├─────────────────┤       ├─────────────────┤       ├─────────────────┤
│ PK id           │       │ PK id           │───┐   │ PK id           │
│ UQ username     │       │ UQ patient_id   │   └───│ FK patient_id   │
│ UQ email        │       │    first_name   │       │    patient_name │
│    password_hash│       │    last_name    │       │    patient_email│
│    full_name    │       │    dob          │       │    patient_phone│
│    role         │       │    gender       │       │    appt_date    │
│    phone        │       │    phone        │       │    appt_time    │
│    is_active    │       │    email        │       │    doctor       │
│    created_at   │       │    address      │       │    department   │
│    last_login   │       │    emergency_   │       │    reason       │
└─────────────────┘       │      contact    │       │    status       │
                          │    emergency_   │       │    notes        │
                          │      phone      │       │    created_at   │
                          │    blood_type   │       │    updated_at   │
                          │    allergies    │       └─────────────────┘
                          │    medical_notes│
                          │    created_at   │
                          │    updated_at   │
                          └─────────────────┘
```

#### 6.3.2 Table Definitions

**Users Table:**
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(120) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(100) NOT NULL,
    role VARCHAR(20) DEFAULT 'nurse' CHECK (role IN ('admin', 'doctor', 'nurse', 'receptionist')),
    phone VARCHAR(20),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP
);

-- Create indexes for performance
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);

-- Add comments
COMMENT ON TABLE users IS 'Staff users who can log into the system';
COMMENT ON COLUMN users.role IS 'User role: admin, doctor, nurse, or receptionist';
```

**Patients Table:**
```sql
CREATE TABLE patients (
    id SERIAL PRIMARY KEY,
    patient_id VARCHAR(20) UNIQUE NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    date_of_birth DATE,
    gender VARCHAR(10) CHECK (gender IN ('Male', 'Female', 'Other')),
    phone VARCHAR(20),
    email VARCHAR(120),
    address TEXT,
    emergency_contact VARCHAR(100),
    emergency_phone VARCHAR(20),
    blood_type VARCHAR(5) CHECK (blood_type IN ('A+', 'A-', 'B+', 'B-', 'AB+', 'AB-', 'O+', 'O-')),
    allergies TEXT,
    medical_notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create indexes
CREATE INDEX idx_patients_patient_id ON patients(patient_id);
CREATE INDEX idx_patients_last_name ON patients(last_name);
CREATE INDEX idx_patients_phone ON patients(phone);

-- Create trigger to update updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_patients_updated_at BEFORE UPDATE ON patients
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Add comments
COMMENT ON TABLE patients IS 'Patient demographic and medical information';
COMMENT ON COLUMN patients.patient_id IS 'Unique patient identifier (e.g., P00001)';
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
    department VARCHAR(50) CHECK (department IN ('General', 'Cardiology', 'Pediatrics', 'Orthopedics', 'Emergency')),
    reason TEXT NOT NULL,
    status VARCHAR(20) DEFAULT 'pending' CHECK (status IN ('pending', 'confirmed', 'completed', 'cancelled')),
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create indexes
CREATE INDEX idx_appointments_patient_id ON appointments(patient_id);
CREATE INDEX idx_appointments_date ON appointments(appointment_date);
CREATE INDEX idx_appointments_status ON appointments(status);
CREATE INDEX idx_appointments_doctor ON appointments(doctor);

-- Create trigger to update updated_at
CREATE TRIGGER update_appointments_updated_at BEFORE UPDATE ON appointments
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Add comments
COMMENT ON TABLE appointments IS 'Patient appointment bookings';
COMMENT ON COLUMN appointments.status IS 'Appointment status: pending, confirmed, completed, or cancelled';
```

**Audit Logs Table (Optional but Recommended):**
```sql
CREATE TABLE audit_logs (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE SET NULL,
    action VARCHAR(50) NOT NULL,
    table_name VARCHAR(50),
    record_id INTEGER,
    old_values JSONB,
    new_values JSONB,
    ip_address VARCHAR(45),
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create indexes
CREATE INDEX idx_audit_logs_user_id ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_action ON audit_logs(action);
CREATE INDEX idx_audit_logs_table_name ON audit_logs(table_name);
CREATE INDEX idx_audit_logs_created_at ON audit_logs(created_at);

COMMENT ON TABLE audit_logs IS 'Audit trail of all user actions';
```

#### 6.3.3 Initialize Database with Sample Data

**Connect to Database:**
```bash
psql -h localhost -U webadmin -d healthclinic -W
```

**Insert Default Users:**
```sql
-- Admin user
INSERT INTO users (username, email, password_hash, full_name, role, phone) VALUES
('admin', 'admin@healthclinic.local', 'scrypt:32768:8:1$XYZ...', 'Administrator', 'admin', '204-555-0100');

-- Doctors
INSERT INTO users (username, email, password_hash, full_name, role, phone) VALUES
('dr.johnson', 'dr.johnson@healthclinic.local', 'scrypt:32768:8:1$ABC...', 'Dr. Sarah Johnson', 'doctor', '204-555-0101'),
('dr.smith', 'dr.smith@healthclinic.local', 'scrypt:32768:8:1$DEF...', 'Dr. Michael Smith', 'doctor', '204-555-0102');

-- Nurses
INSERT INTO users (username, email, password_hash, full_name, role, phone) VALUES
('nurse.maria', 'nurse.maria@healthclinic.local', 'scrypt:32768:8:1$GHI...', 'Maria Garcia', 'nurse', '204-555-0103'),
('nurse.jennifer', 'nurse.jennifer@healthclinic.local', 'scrypt:32768:8:1$JKL...', 'Jennifer Lee', 'nurse', '204-555-0104');

-- Verify
SELECT username, full_name, role FROM users;
```

**Insert Sample Patients:**
```sql
-- Sample patient 1
INSERT INTO patients (patient_id, first_name, last_name, date_of_birth, gender, phone, email, address, blood_type) VALUES
('P00001', 'John', 'Doe', '1985-05-15', 'Male', '204-555-1001', 'john.doe@email.com', '123 Main St, Winnipeg, MB', 'O+');

-- Sample patient 2
INSERT INTO patients (patient_id, first_name, last_name, date_of_birth, gender, phone, email, address, blood_type, allergies) VALUES
('P00002', 'Jane', 'Smith', '1990-08-22', 'Female', '204-555-1002', 'jane.smith@email.com', '456 Oak Ave, Winnipeg, MB', 'A+', 'Penicillin');

-- Sample patient 3
INSERT INTO patients (patient_id, first_name, last_name, date_of_birth, gender, phone, email, address, blood_type) VALUES
('P00003', 'Robert', 'Brown', '1975-12-10', 'Male', '204-555-1003', 'robert.brown@email.com', '789 Elm St, Winnipeg, MB', 'B-');

-- Verify
SELECT patient_id, first_name, last_name, phone FROM patients;
```

**Insert Sample Appointments:**
```sql
-- Upcoming appointments
INSERT INTO appointments (patient_id, patient_name, patient_email, patient_phone, appointment_date, appointment_time, doctor, department, reason, status) VALUES
(1, 'John Doe', 'john.doe@email.com', '204-555-1001', CURRENT_DATE + INTERVAL '2 days', '10:00 AM', 'Dr. Sarah Johnson', 'General', 'Annual checkup', 'confirmed'),
(2, 'Jane Smith', 'jane.smith@email.com', '204-555-1002', CURRENT_DATE + INTERVAL '3 days', '2:30 PM', 'Dr. Michael Smith', 'Cardiology', 'Follow-up consultation', 'confirmed'),
(3, 'Robert Brown', 'robert.brown@email.com', '204-555-1003', CURRENT_DATE + INTERVAL '1 day', '9:00 AM', 'Dr. Sarah Johnson', 'General', 'Blood pressure check', 'pending');

-- Verify
SELECT patient_name, appointment_date, appointment_time, doctor, status FROM appointments;
```

### 6.4 Flask SQLAlchemy Integration

#### 6.4.1 Database Connection Configuration

**File: `/var/www/healthclinic/app/__init__.py`**
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager

# Initialize extensions
db = SQLAlchemy()
login_manager = LoginManager()

def create_app():
    """Application factory pattern"""
    app = Flask(__name__)
    
    # Secret key for session management
    app.config['SECRET_KEY'] = 'healthclinic-secret-key-2025'
    
    # PostgreSQL Database URI
    # Format: postgresql://username:password@host:port/database
    # Note: @ symbol in password must be URL-encoded as %40
    app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://webadmin:T%40ylorSwift13@127.0.0.1:5432/healthclinic'
    
    # Disable track modifications (improves performance)
    app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
    
    # Database pool configuration
    app.config['SQLALCHEMY_POOL_SIZE'] = 10
    app.config['SQLALCHEMY_POOL_TIMEOUT'] = 30
    app.config['SQLALCHEMY_POOL_RECYCLE'] = 3600
    app.config['SQLALCHEMY_MAX_OVERFLOW'] = 20
    
    # Echo SQL queries (disable in production)
    app.config['SQLALCHEMY_ECHO'] = False
    
    # Initialize extensions with app
    db.init_app(app)
    login_manager.init_app(app)
    login_manager.login_view = 'main.login'
    login_manager.login_message = 'Please log in to access this page.'
    
    # Register blueprints
    from .routes import main_bp
    app.register_blueprint(main_bp)
    
    return app
```

#### 6.4.2 Database Models (ORM)

**File: `/var/www/healthclinic/app/models.py`**
```python
from . import db, login_manager
from flask_login import UserMixin
from werkzeug.security import generate_password_hash, check_password_hash
from datetime import datetime

@login_manager.user_loader
def load_user(user_id):
    """Load user by ID for Flask-Login"""
    return User.query.get(int(user_id))

class User(UserMixin, db.Model):
    """User model for staff authentication"""
    __tablename__ = 'users'
    
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(50), unique=True, nullable=False, index=True)
    email = db.Column(db.String(120), unique=True, nullable=False, index=True)
    password_hash = db.Column(db.String(255), nullable=False)
    full_name = db.Column(db.String(100), nullable=False)
    role = db.Column(db.String(20), default='nurse', index=True)
    phone = db.Column(db.String(20))
    is_active = db.Column(db.Boolean, default=True)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    last_login = db.Column(db.DateTime)
    
    def set_password(self, password):
        """Hash and set password"""
        self.password_hash = generate_password_hash(password)
    
    def check_password(self, password):
        """Verify password"""
        return check_password_hash(self.password_hash, password)
    
    def __repr__(self):
        return f'<User {self.username}>'

class Patient(db.Model):
    """Patient demographic and medical information"""
    __tablename__ = 'patients'
    
    id = db.Column(db.Integer, primary_key=True)
    patient_id = db.Column(db.String(20), unique=True, nullable=False, index=True)
    first_name = db.Column(db.String(50), nullable=False)
    last_name = db.Column(db.String(50), nullable=False, index=True)
    date_of_birth = db.Column(db.Date)
    gender = db.Column(db.String(10))
    phone = db.Column(db.String(20), index=True)
    email = db.Column(db.String(120))
    address = db.Column(db.Text)
    emergency_contact = db.Column(db.String(100))
    emergency_phone = db.Column(db.String(20))
    blood_type = db.Column(db.String(5))
    allergies = db.Column(db.Text)
    medical_notes = db.Column(db.Text)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    
    # Relationship to appointments
    appointments = db.relationship('Appointment', backref='patient', lazy='dynamic', cascade='all, delete-orphan')
    
    @property
    def full_name(self):
        """Return full name"""
        return f"{self.first_name} {self.last_name}"
    
    @property
    def age(self):
        """Calculate age from date of birth"""
        if self.date_of_birth:
            today = datetime.today().date()
            return today.year - self.date_of_birth.year - ((today.month, today.day) < (self.date_of_birth.month, self.date_of_birth.day))
        return None
    
    def __repr__(self):
        return f'<Patient {self.patient_id}: {self.full_name}>'

class Appointment(db.Model):
    """Patient appointment bookings"""
    __tablename__ = 'appointments'
    
    id = db.Column(db.Integer, primary_key=True)
    patient_id = db.Column(db.Integer, db.ForeignKey('patients.id', ondelete='CASCADE'), index=True)
    patient_name = db.Column(db.String(100), nullable=False)
    patient_email = db.Column(db.String(120))
    patient_phone = db.Column(db.String(20))
    appointment_date = db.Column(db.Date, nullable=False, index=True)
    appointment_time = db.Column(db.String(10), nullable=False)
    doctor = db.Column(db.String(50), nullable=False, index=True)
    department = db.Column(db.String(50))
    reason = db.Column(db.Text, nullable=False)
    status = db.Column(db.String(20), default='pending', index=True)
    notes = db.Column(db.Text)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    
    @property
    def is_upcoming(self):
        """Check if appointment is in the future"""
        return self.appointment_date >= datetime.today().date()
    
    def __repr__(self):
        return f'<Appointment {self.id}: {self.patient_name} on {self.appointment_date}>'

class AuditLog(db.Model):
    """Audit trail of user actions"""
    __tablename__ = 'audit_logs'
    
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('users.id', ondelete='SET NULL'), index=True)
    action = db.Column(db.String(50), nullable=False, index=True)
    table_name = db.Column(db.String(50), index=True)
    record_id = db.Column(db.Integer)
    old_values = db.Column(db.JSON)
    new_values = db.Column(db.JSON)
    ip_address = db.Column(db.String(45))
    user_agent = db.Column(db.Text)
    created_at = db.Column(db.DateTime, default=datetime.utcnow, index=True)
    
    def __repr__(self):
        return f'<AuditLog {self.id}: {self.action} by User {self.user_id}>'
```

#### 6.4.3 Database Initialization Script

**File: `/var/www/healthclinic/init_db.py`**
```python
#!/usr/bin/env python3
"""
Database initialization script for Healthcare Clinic Portal
Creates all tables and optionally adds sample data
"""

from app import create_app, db
from app.models import User, Patient, Appointment
from datetime import datetime, timedelta

def init_database(sample_data=False):
    """Initialize database tables"""
    app = create_app()
    
    with app.app_context():
        print("Creating database tables...")
        db.create_all()
        print(" Database tables created successfully!")
        
        if sample_data:
            print("\nAdding sample data...")
            add_sample_data()
            print(" Sample data added successfully!")
        
        # Verify tables
        print("\nVerifying tables:")
        inspector = db.inspect(db.engine)
        tables = inspector.get_table_names()
        print(f"Found {len(tables)} tables:")
        for table in tables:
            print(f"  - {table}")

def add_sample_data():
    """Add sample users, patients, and appointments"""
    
    # Create admin user
    admin = User(
        username='admin',
        email='admin@healthclinic.local',
        full_name='System Administrator',
        role='admin',
        phone='204-555-0100'
    )
    admin.set_password('g3company!@#')
    db.session.add(admin)
    
    # Create doctor users
    dr_johnson = User(
        username='dr.johnson',
        email='dr.johnson@healthclinic.local',
        full_name='Dr. Sarah Johnson',
        role='doctor',
        phone='204-555-0101'
    )
    dr_johnson.set_password('g3company!@#')
    db.session.add(dr_johnson)
    
    dr_smith = User(
        username='dr.smith',
        email='dr.smith@healthclinic.local',
        full_name='Dr. Michael Smith',
        role='doctor',
        phone='204-555-0102'
    )
    dr_smith.set_password('g3company!@#')
    db.session.add(dr_smith)
    
    # Create nurse users
    nurse_maria = User(
        username='nurse.maria',
        email='nurse.maria@healthclinic.local',
        full_name='Maria Garcia',
        role='nurse',
        phone='204-555-0103'
    )
    nurse_maria.set_password('g3company!@#')
    db.session.add(nurse_maria)
    
    db.session.commit()
    print("  ✓ Created 4 staff users")
    
    # Create sample patients
    patient1 = Patient(
        patient_id='P00001',
        first_name='John',
        last_name='Doe',
        date_of_birth=datetime(1985, 5, 15).date(),
        gender='Male',
        phone='204-555-1001',
        email='john.doe@email.com',
        address='123 Main St, Winnipeg, MB',
        blood_type='O+'
    )
    db.session.add(patient1)
    
    patient2 = Patient(
        patient_id='P00002',
        first_name='Jane',
        last_name='Smith',
        date_of_birth=datetime(1990, 8, 22).date(),
        gender='Female',
        phone='204-555-1002',
        email='jane.smith@email.com',
        address='456 Oak Ave, Winnipeg, MB',
        blood_type='A+',
        allergies='Penicillin'
    )
    db.session.add(patient2)
    
    patient3 = Patient(
        patient_id='P00003',
        first_name='Robert',
        last_name='Brown',
        date_of_birth=datetime(1975, 12, 10).date(),
        gender='Male',
        phone='204-555-1003',
        email='robert.brown@email.com',
        address='789 Elm St, Winnipeg, MB',
        blood_type='B-'
    )
    db.session.add(patient3)
    
    db.session.commit()
    print("  ✓ Created 3 sample patients")
    
    # Create sample appointments
    today = datetime.today().date()
    
    appt1 = Appointment(
        patient_id=1,
        patient_name='John Doe',
        patient_email='john.doe@email.com',
        patient_phone='204-555-1001',
        appointment_date=today + timedelta(days=2),
        appointment_time='10:00 AM',
        doctor='Dr. Sarah Johnson',
        department='General',
        reason='Annual checkup',
        status='confirmed'
    )
    db.session.add(appt1)
    
    appt2 = Appointment(
        patient_id=2,
        patient_name='Jane Smith',
        patient_email='jane.smith@email.com',
        patient_phone='204-555-1002',
        appointment_date=today + timedelta(days=3),
        appointment_time='2:30 PM',
        doctor='Dr. Michael Smith',
        department='Cardiology',
        reason='Follow-up consultation',
        status='confirmed'
    )
    db.session.add(appt2)
    
    appt3 = Appointment(
        patient_id=3,
        patient_name='Robert Brown',
        patient_email='robert.brown@email.com',
        patient_phone='204-555-1003',
        appointment_date=today + timedelta(days=1),
        appointment_time='9:00 AM',
        doctor='Dr. Sarah Johnson',
        department='General',
        reason='Blood pressure check',
        status='pending'
    )
    db.session.add(appt3)
    
    db.session.commit()
    print("  ✓ Created 3 sample appointments")

if __name__ == '__main__':
    import sys
    
    # Check if user wants sample data
    sample_data = '--sample' in sys.argv or '-s' in sys.argv
    
    init_database(sample_data=sample_data)
    
    print("\n" + "="*60)
    print("DATABASE INITIALIZATION COMPLETE!")
    print("="*60)
    print("\nDefault credentials:")
    print("  Username: admin")
    print("  Password: g3company!@#")
    print("\nWeb application: http://10.10.40.30")
    print("="*60)
```

**Run Database Initialization:**
```bash
# Navigate to project directory
cd /var/www/healthclinic

# Activate virtual environment
source venv/bin/activate

# Make script executable
chmod +x init_db.py

# Initialize database with sample data
python3 init_db.py --sample

# Expected output:
# Creating database tables...
#  Database tables created successfully!
# 
# Adding sample data...
#   ✓ Created 4 staff users
#   ✓ Created 3 sample patients
#   ✓ Created 3 sample appointments
#  Sample data added successfully!
# 
# Verifying tables:
# Found 4 tables:
#   - users
#   - patients
#   - appointments
#   - audit_logs
# 
# ============================================================
# DATABASE INITIALIZATION COMPLETE!
# ============================================================
# 
# Default credentials:
#   Username: admin
#   Password: g3company!@#
# 
# Web application: http://10.10.40.30
# ============================================================
```

### 6.5 pgAdmin Setup (Database GUI Tool)

#### 6.5.1 Install Docker and Docker Compose

**On WEB01 or separate management machine:**
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add current user to docker group
sudo usermod -aG docker $USER

# Log out and back in, or run:
newgrp docker

# Verify Docker installation
docker --version
```

#### 6.5.2 Deploy pgAdmin Container

**Create pgAdmin directory:**
```bash
mkdir -p ~/pgadmin
cd ~/pgadmin
```

**Create Docker Compose file:**
```bash
nano docker-compose.yml
```

**Content:**
```yaml
version: '3.8'

services:
  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: pgadmin
    restart: always
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@healthclinic.local
      PGADMIN_DEFAULT_PASSWORD: Clinic@2025!
      PGADMIN_LISTEN_PORT: 80
    ports:
      - "5050:80"
    volumes:
      - pgadmin_data:/var/lib/pgadmin
    networks:
      - pgadmin_network

volumes:
  pgadmin_data:
    driver: local

networks:
  pgadmin_network:
    driver: bridge
```

**Start pgAdmin:**
```bash
# Start container
docker-compose up -d

# Verify container is running
docker ps

# Check logs
docker-compose logs -f pgadmin
```

**Access pgAdmin:**
```
URL: http://10.10.40.30:5050
Email: admin@healthclinic.local
Password: Clinic@2025!
```

#### 6.5.3 Configure Database Connections in pgAdmin

**Add WEB01 PostgreSQL Server:**

1. Login to pgAdmin
2. Right-click **Servers** → **Register** → **Server**

**General Tab:**
- Name: `WEB01 - Healthcare Clinic DB`

**Connection Tab:**
- Host name/address: `10.10.40.30`
- Port: `5432`
- Maintenance database: `healthclinic`
- Username: `webadmin`
- Password: `T@ylorSwift13`
- Save password: Yes

**SSL Tab:**
- SSL mode: `Prefer`

**Advanced Tab:**
- DB restriction: `healthclinic`

Click **Save**

**Add ZABBIX01 PostgreSQL Server:**

1. Right-click **Servers** → **Register** → **Server**

**General Tab:**
- Name: `ZABBIX01 - Monitoring DB`

**Connection Tab:**
- Host name/address: `10.10.40.40`
- Port: `5432`
- Maintenance database: `zabbix`
- Username: `zabbixuser`
- Password: `g3company!@#`
- Save password: Yes

**SSL Tab:**
- SSL mode: `Prefer`

Click **Save**

### 6.6 Database Maintenance & Operations

#### 6.6.1 Backup Scripts

**PostgreSQL Backup Script:**
```bash
# Create backup directory
sudo mkdir -p /var/backups/postgresql
sudo chown webadmin:webadmin /var/backups/postgresql

# Create backup script
sudo nano /usr/local/bin/backup_postgres.sh
```

**Content:**
```bash
#!/bin/bash
#########################################################
# PostgreSQL Backup Script
# Purpose: Daily backup of Healthcare Clinic databases
# Author: Kristine Bagsican
# Date: December 2025
#########################################################

# Configuration
BACKUP_DIR="/var/backups/postgresql"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

# Database credentials
WEB_HOST="localhost"
WEB_USER="webadmin"
WEB_DB="healthclinic"
PGPASSWORD="T@ylorSwift13"

# Create backup directory if not exists
mkdir -p $BACKUP_DIR

# Backup Healthcare Clinic database
echo "Starting backup of healthclinic database..."
PGPASSWORD=$PGPASSWORD pg_dump -h $WEB_HOST -U $WEB_USER -d $WEB_DB -F c -f "$BACKUP_DIR/healthclinic_${DATE}.backup"

if [ $? -eq 0 ]; then
    echo "  Backup completed: healthclinic_${DATE}.backup"
else
    echo "❌ Backup failed!"
    exit 1
fi

# Compress backup
gzip "$BACKUP_DIR/healthclinic_${DATE}.backup"
echo "  Backup compressed: healthclinic_${DATE}.backup.gz"

# Remove backups older than retention period
find $BACKUP_DIR -name "healthclinic_*.backup.gz" -mtime +$RETENTION_DAYS -delete
echo "  Old backups removed (retention: $RETENTION_DAYS days)"

# Calculate backup size
BACKUP_SIZE=$(du -sh "$BACKUP_DIR/healthclinic_${DATE}.backup.gz" | cut -f1)
echo "Backup size: $BACKUP_SIZE"

echo "===================="
echo "Backup completed successfully!"
echo "File: $BACKUP_DIR/healthclinic_${DATE}.backup.gz"
echo "===================="
```

**Make executable:**
```bash
sudo chmod +x /usr/local/bin/backup_postgres.sh

# Test backup
/usr/local/bin/backup_postgres.sh
```

**Schedule with Cron:**
```bash
# Edit crontab
crontab -e

# Add daily backup at 3:00 AM
0 3 * * * /usr/local/bin/backup_postgres.sh >> /var/log/postgres_backup.log 2>&1
```

#### 6.6.2 Database Restore

**Restore from backup:**
```bash
# List available backups
ls -lh /var/backups/postgresql/

# Restore specific backup
BACKUP_FILE="/var/backups/postgresql/healthclinic_20251216_030000.backup.gz"

# Decompress
gunzip -k $BACKUP_FILE

# Stop web application (to prevent new connections)
sudo systemctl stop apache2

# Drop existing database (CAUTION!)
sudo -u postgres psql -c "DROP DATABASE IF EXISTS healthclinic;"

# Recreate database
sudo -u postgres psql -c "CREATE DATABASE healthclinic OWNER webadmin;"

# Restore backup
pg_restore -h localhost -U webadmin -d healthclinic -c /var/backups/postgresql/healthclinic_20251216_030000.backup

# Restart web application
sudo systemctl start apache2

echo "  Database restored successfully!"
```

#### 6.6.3 Database Maintenance Commands

**Vacuum and Analyze (Performance optimization):**
```bash
# Connect to database
psql -h localhost -U webadmin -d healthclinic -W
```
```sql
-- Vacuum all tables (reclaim storage)
VACUUM;

-- Vacuum with analyze (update statistics)
VACUUM ANALYZE;

-- Vacuum specific table
VACUUM ANALYZE patients;

-- Full vacuum (more aggressive, locks table)
VACUUM FULL appointments;

-- Check database size
SELECT pg_database.datname, pg_size_pretty(pg_database_size(pg_database.datname)) AS size 
FROM pg_database 
WHERE datname = 'healthclinic';

-- Check table sizes
SELECT 
    schemaname AS schema,
    tablename AS table,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size,
    pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) AS table_size,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename) - pg_relation_size(schemaname||'.'||tablename)) AS index_size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Check index usage
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan AS index_scans,
    idx_tup_read AS tuples_read,
    idx_tup_fetch AS tuples_fetched
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY idx_scan ASC;

\q
```

**Reindex (Rebuild indexes):**
```sql
-- Reindex all tables
REINDEX DATABASE healthclinic;

-- Reindex specific table
REINDEX TABLE patients;

-- Reindex specific index
REINDEX INDEX idx_patients_patient_id;
```

#### 6.6.4 Monitoring Database Performance

**Check active connections:**
```sql
-- View current connections
SELECT 
    pid,
    usename AS user,
    application_name,
    client_addr,
    state,
    query_start,
    state_change,
    wait_event_type,
    query
FROM pg_stat_activity
WHERE datname = 'healthclinic'
ORDER BY query_start DESC;

-- Count connections by state
SELECT state, COUNT(*) 
FROM pg_stat_activity 
WHERE datname = 'healthclinic'
GROUP BY state;

-- Kill a specific connection (if needed)
-- SELECT pg_terminate_backend(pid) WHERE pid = 12345;
```

**Check slow queries:**
```sql
-- Enable query logging (add to postgresql.conf)
-- log_statement = 'all'
-- log_duration = on
-- log_min_duration_statement = 1000  # Log queries > 1 second

-- View slow queries from log
SELECT 
    query,
    calls,
    total_time,
    mean_time,
    max_time
FROM pg_stat_statements
WHERE query NOT LIKE '%pg_%'
ORDER BY mean_time DESC
LIMIT 20;
```

**Check database statistics:**
```sql
-- Table statistics
SELECT 
    schemaname,
    relname AS table_name,
    n_tup_ins AS inserts,
    n_tup_upd AS updates,
    n_tup_del AS deletes,
    n_live_tup AS live_rows,
    n_dead_tup AS dead_rows,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
WHERE schemaname = 'public'
ORDER BY n_tup_ins + n_tup_upd + n_tup_del DESC;
```

### 6.7 Database Security Best Practices

#### 6.7.1 Password Management

**Change database passwords:**
```sql
-- Change webadmin password
ALTER ROLE webadmin WITH PASSWORD 'NewSecurePassword2025!';

-- Change zabbixuser password
ALTER ROLE zabbixuser WITH PASSWORD 'NewZabbixPass2025!';

-- After changing, update application configs:
-- - /var/www/healthclinic/app/__init__.py
-- - /etc/zabbix/zabbix_server.conf
```

#### 6.7.2 Access Control

**Create read-only user for reporting:**
```sql
-- Create read-only user
CREATE ROLE readonly WITH LOGIN PASSWORD 'ReadOnly2025!';

-- Grant connect privilege
GRANT CONNECT ON DATABASE healthclinic TO readonly;

-- Grant usage on schema
GRANT USAGE ON SCHEMA public TO readonly;

-- Grant select on all tables
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;

-- Grant select on future tables
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO readonly;

-- Verify permissions
\du readonly
```

#### 6.7.3 SSL/TLS Encryption

**Enable SSL for PostgreSQL:**
```bash
# Generate self-signed certificate
sudo openssl req -new -x509 -days 365 -nodes -text \
    -out /etc/postgresql/15/main/server.crt \
    -keyout /etc/postgresql/15/main/server.key \
    -subj "/CN=healthclinic.local"

# Set permissions
sudo chown postgres:postgres /etc/postgresql/15/main/server.crt
sudo chown postgres:postgres /etc/postgresql/15/main/server.key
sudo chmod 600 /etc/postgresql/15/main/server.key

# Edit postgresql.conf
sudo nano /etc/postgresql/15/main/postgresql.conf

# Enable SSL
ssl = on
ssl_cert_file = '/etc/postgresql/15/main/server.crt'
ssl_key_file = '/etc/postgresql/15/main/server.key'

# Restart PostgreSQL
sudo systemctl restart postgresql
```

**Force SSL connections:**
```bash
# Edit pg_hba.conf
sudo nano /etc/postgresql/15/main/pg_hba.conf

# Change 'md5' to 'scram-sha-256' and add 'hostssl' instead of 'host':
hostssl    all    all    10.10.40.0/24    scram-sha-256

# Restart PostgreSQL
sudo systemctl restart postgresql
```

### 6.8 Database Troubleshooting

#### 6.8.1 Connection Issues

**Cannot connect to PostgreSQL:**
```bash
# Check if PostgreSQL is running
sudo systemctl status postgresql

# Check if port 5432 is listening
sudo ss -tlnp | grep 5432

# Test local connection
psql -h localhost -U webadmin -d healthclinic -W

# Check pg_hba.conf
sudo nano /etc/postgresql/15/main/pg_hba.conf

# Check postgresql.conf
sudo nano /etc/postgresql/15/main/postgresql.conf

# View PostgreSQL logs
sudo tail -100 /var/log/postgresql/postgresql-15-main.log

# Restart PostgreSQL
sudo systemctl restart postgresql
```

#### 6.8.2 Performance Issues

**Database is slow:**
```sql
-- Check for locks
SELECT 
    pid,
    usename,
    pg_blocking_pids(pid) AS blocked_by,
    query AS blocked_query
FROM pg_stat_activity
WHERE cardinality(pg_blocking_pids(pid)) > 0;

-- Check for long-running queries
SELECT 
    pid,
    now() - query_start AS duration,
    usename,
    query
FROM pg_stat_activity
WHERE state != 'idle'
AND now() - query_start > interval '5 minutes'
ORDER BY duration DESC;

-- Terminate long-running query
-- SELECT pg_terminate_backend(pid);

-- Check cache hit ratio (should be > 99%)
SELECT 
    sum(heap_blks_read) AS heap_read,
    sum(heap_blks_hit) AS heap_hit,
    sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) AS cache_hit_ratio
FROM pg_statio_user_tables;
```

#### 6.8.3 Disk Space Issues

**Database running out of space:**
```bash
# Check disk usage
df -h /var/lib/postgresql

# Check database sizes
psql -h localhost -U webadmin -d healthclinic -W -c "SELECT pg_database.datname, pg_size_pretty(pg_database_size(pg_database.datname)) AS size FROM pg_database;"

# Vacuum to reclaim space
psql -h localhost -U webadmin -d healthclinic -W -c "VACUUM FULL;"

# Remove old backups
find /var/backups/postgresql -name "*.backup.gz" -mtime +30 -delete

# Check largest tables
psql -h localhost -U webadmin -d healthclinic -W -c "SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size FROM pg_tables WHERE schemaname = 'public' ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC LIMIT 10;"
```

---

## 7. Security Implementation

### 7.1 Network Security

#### 7.1.1 VLAN Segmentation & Access Control

**VLAN Access Matrix:**

| Source VLAN | Destination | Allowed Services | Blocked Services |
|-------------|-------------|------------------|------------------|
| **VLAN 10** (Patient WiFi) | DC01/DC02 | DNS (53) | RDP (3389), SMB (445) |
| **VLAN 10** (Patient WiFi) | Internet | HTTP/HTTPS | All internal servers |
| **VLAN 20** (Clinical Staff) | DC01/DC02 | DNS, RDP, SMB | - |
| **VLAN 20** (Clinical Staff) | WEB01 | HTTP (80), HTTPS (443) | SSH (22) |
| **VLAN 20** (Clinical Staff) | MAIL01 | SMTP, IMAP, Webmail | SSH (22) |
| **VLAN 20** (Clinical Staff) | TrueNAS | SMB (445) | SSH (22) |
| **VLAN 30** (Admin) | All Servers | All Services | - |
| **VLAN 40** (IT/Servers) | All | All | - |

**PostgreSQL Firewall Rules:**
```bash
# On WEB01:
sudo ufw allow from 10.10.40.0/24 to any port 5432 comment 'PostgreSQL from server VLAN'
sudo ufw allow from 10.10.30.0/24 to any port 5432 comment 'PostgreSQL from admin VLAN'
sudo ufw allow from 172.17.0.0/16 to any port 5432 comment 'PostgreSQL from Docker (pgAdmin)'
sudo ufw deny 5432 comment 'Block PostgreSQL from other networks'
```

---

## 12. Appendices

### Appendix A: All Credentials

| System | Username | Password | Notes |
|--------|----------|----------|-------|
| **Domain** | Administrator | Clinic@2025! | Domain admin |
| **DC01/DC02** | Administrator | Clinic@2025! | Local admin |
| **Cisco Devices** | admin | Clinic@2025! | Console/SSH |
| **PostgreSQL (WEB01)** | webadmin | T@ylorSwift13 | Superuser |
| **PostgreSQL (ZABBIX)** | zabbixuser | g3company!@# | Zabbix DB user |
| **Flask Web App** | admin | g3company!@# | Web portal admin |
| **pgAdmin** | admin@healthclinic.local | Clinic@2025! | Database GUI |
| **Zabbix Web** | Admin | Clinic@2025! | Monitoring GUI |
| **Mailcow** | admin | moohoo | Email admin |
| **TrueNAS** | root | Clinic@2025! | Storage admin |
| **Tailscale** | (Use Google/MS account) | - | VPN service |

### Appendix B: URLs & Access Points

| Service | URL | Port | Access From |
|---------|-----|------|-------------|
| **Healthcare Portal** | http://10.10.40.30 | 80 | All VLANs |
| **pgAdmin** | http://10.10.40.30:5050 | 5050 | VLAN 30, 40 |
| **Mailcow Webmail** | https://10.10.40.20 | 443 | VLAN 20, 30, 40 |
| **Zabbix Dashboard** | http://10.10.40.40/zabbix | 80 | VLAN 30, 40 |
| **TrueNAS UI** | http://10.10.40.50 | 80 | VLAN 30, 40 |
| **DC01 RDP** | 10.10.40.10:3389 | 3389 | VLAN 30, 40 |
| **DC02 RDP** | 10.10.40.11:3389 | 3389 | VLAN 30, 40 |

### Appendix C: Project Team

**Team:** Kristine Bagsican, Swathi Anil, Kunwei Li, Jaered Bacolod
**Institution:** Manitoba Institute of Trades and Technology (M.I.T.T.)  
**Program:** Network and Systems Administrator Diploma  
**Location:** Winnipeg, Manitoba, Canada  
**Completion Date:** December 2025

### Appendix D: References

- PostgreSQL Official Documentation: https://www.postgresql.org/docs/
- Flask-SQLAlchemy Documentation: https://flask-sqlalchemy.palletsprojects.com/
- Zabbix Documentation: https://www.zabbix.com/documentation/current/
- TrueNAS SCALE Documentation: https://www.truenas.com/docs/scale/
- Cisco IOS Documentation: https://www.cisco.com/c/en/us/support/ios-nx-os-software/
- Tailscale Documentation: https://tailscale.com/kb/

---

**Document Version:** 1.0  
**Last Updated:** December 16, 2025  
**Prepared By:** Kristine Bagsican & Team  
**Status:** Production Ready

---

*This documentation is comprehensive and production-ready. All systems are operational and tested.*


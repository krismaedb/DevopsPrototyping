# 🌐 WEB01 Complete Setup Guide - Healthcare Clinic Project

**Complete step-by-step guide for WEB01 (Web Application Server)**  
**Manitoba Institute of Trades and Technology (M.I.T.T.) - Network & Systems Administrator Capstone**

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [VM Specifications](#vm-specifications)
- [Part 1: VM Creation in Proxmox](#part-1-vm-creation-in-proxmox)
- [Part 2: Ubuntu Server Installation](#part-2-ubuntu-server-installation)
- [Part 3: Network Configuration](#part-3-network-configuration)
- [Part 4: Desktop GUI Installation](#part-4-desktop-gui-installation)
- [Part 5: Web Server Setup](#part-5-web-server-setup)
- [Part 6: Database Configuration](#part-6-database-configuration)
- [Part 7: Flask Application Setup](#part-7-flask-application-setup)
- [Part 8: Apache WSGI Configuration](#part-8-apache-wsgi-configuration)
- [Part 9: Testing & Verification](#part-9-testing--verification)
- [Part 10: Troubleshooting](#part-10-troubleshooting)
- [Completion Checklist](#completion-checklist)
- [Quick Reference](#quick-reference)

---

## Project Overview

**WEB01** is a web application server hosting a patient portal and appointment booking system for Healthcare Clinic.

### Technology Stack
- **Operating System:** Ubuntu 24.04 LTS Server
- **Desktop Environment:** GNOME Desktop (minimal)
- **Web Server:** Apache 2.4
- **Application Framework:** Flask (Python)
- **Database:** PostgreSQL 15
- **WSGI Server:** mod_wsgi

### Network Details
- **VM Name:** WEB01
- **IP Address:** 10.10.40.30/24
- **Gateway:** 10.10.40.1 (HSRP VIP on R1/R2)
- **DNS Servers:** 10.10.40.10 (DC01), 10.10.40.11 (DC02)
- **VLAN:** 40 (IT/Servers)
- **Domain:** healthclinic.local

---

## VM Specifications
```yaml
Proxmox Configuration:
  VM ID: 130
  Name: WEB01
  Node: pve-main (10.10.40.15)
  
Hardware:
  CPU: 4 cores (host type)
  RAM: 8192 MB (8 GB)
  Disk: 80 GB (local-lvm storage)
  Network: VirtIO on vmbr0 (VLAN 40)
  
Operating System:
  Distribution: Ubuntu 24.04.1 LTS
  Architecture: x86_64 (amd64)
  Kernel: 6.8.x
  Desktop: GNOME (minimal)
  
Network Configuration:
  IP: 10.10.40.30/24 (static)
  Gateway: 10.10.40.1
  DNS: 10.10.40.10, 10.10.40.11
  Hostname: web01.healthclinic.local
```

---

## Part 1: VM Creation in Proxmox

### Step 1.1: Access Proxmox Web Interface
```
URL: https://10.10.40.15:8006
Username: root
Password: Clinic@2025!
```

### Step 1.2: Upload Ubuntu ISO (if not already uploaded)
```bash
# Via Proxmox console (SSH or direct):
cd /var/lib/vz/template/iso/

# Download Ubuntu 24.04 Server ISO
wget https://releases.ubuntu.com/24.04/ubuntu-24.04.1-live-server-amd64.iso

# Verify download
ls -lh ubuntu-24.04.1-live-server-amd64.iso
```

**Or upload via Web UI:**
1. Click **pve-main** → **local** → **ISO Images**
2. Click **Upload** or **Download from URL**
3. Select/download `ubuntu-24.04.1-live-server-amd64.iso`

### Step 1.3: Create New VM

Click **Create VM** button (top-right, blue button)

#### Tab 1: General
```
Node: pve-main
VM ID: 130
Name: WEB01
Resource Pool: (leave blank)
☑ Start at boot
```
Click **Next**

#### Tab 2: OS
```
☑ Use CD/DVD disc image file (iso)
Storage: local
ISO image: ubuntu-24.04.1-live-server-amd64.iso

Guest OS:
  Type: Linux
  Version: 6.x - 2.6 Kernel
```
Click **Next**

#### Tab 3: System
```
Graphic card: Default
Machine: Default (i440fx)
BIOS: Default (SeaBIOS)
SCSI Controller: VirtIO SCSI single
☑ Qemu Agent
☐ Add TPM
```
Click **Next**

#### Tab 4: Disks
```
Bus/Device: SCSI 0
Storage: local-lvm
Disk size (GiB): 80
Cache: Default (No cache)
☑ Discard
☐ SSD emulation
☑ IO thread
```
Click **Next**

#### Tab 5: CPU
```
Sockets: 1
Cores: 4
Type: host
```
Click **Next**

#### Tab 6: Memory
```
Memory (MiB): 8192
Minimum memory (MiB): 2048
☑ Ballooning Device
```
Click **Next**

#### Tab 7: Network
```
Bridge: vmbr0
VLAN Tag: (leave blank)
Model: VirtIO (paravirtualized)
☐ Firewall
MAC address: auto
```
Click **Next**

#### Tab 8: Confirm
```
☑ Start after created
```
Click **Finish**

---

## Part 2: Ubuntu Server Installation

### Step 2.1: Open VM Console

1. Click **WEB01 (130)** in left panel
2. Click **Console** button
3. VM boots from Ubuntu ISO

### Step 2.2: Installation Wizard

#### Language Selection
```
> English
```
Press **Enter**

#### Installer Update
```
> Continue without updating
```
Press **Enter**

#### Keyboard Configuration
```
Layout: English (US)
Variant: English (US)
```
Press **Enter**

#### Installation Type
```
> Ubuntu Server (minimized)
```
Press **Enter**

#### Network Connections
```
ens18 eth DHCPv4 10.10.40.xxx/24
(temporary DHCP)
```
Press **Enter**

#### Proxy Configuration
```
Proxy address: (leave blank)
```
Press **Enter**

#### Ubuntu Archive Mirror
```
Mirror: http://archive.ubuntu.com/ubuntu
```
Press **Enter**

#### Guided Storage Configuration
```
☑ Use an entire disk
☑ Set up this disk as an LVM group
Disk: SCSI 0 (80GB)
```
Press **Enter** → **Continue**

#### Profile Setup
```
Your name: Web Admin
Your server's name: web01
Pick a username: webadmin
Choose a password: Clinic@2025!
Confirm your password: Clinic@2025!
```
Press **Enter**

#### SSH Setup
```
☑ Install OpenSSH server
Import SSH identity: No
```
Press **Enter**

#### Featured Server Snaps
```
(Do not select any)
```
Press **Enter**

### Step 2.3: Wait for Installation (10-15 minutes)

### Step 2.4: Reboot
```
Select: Reboot Now
Press Enter when prompted
```

---

## Part 3: Network Configuration

### Step 3.1: First Login
```
web01 login: webadmin
Password: Clinic@2025!
```

### Step 3.2: Configure Static IP
```bash
# Edit netplan
sudo nano /etc/netplan/50-cloud-init.yaml
```

**Delete all content and paste:**
```yaml
network:
  version: 2
  ethernets:
    ens18:
      addresses:
        - 10.10.40.30/24
      routes:
        - to: default
          via: 10.10.40.1
      nameservers:
        addresses:
          - 10.10.40.10
          - 10.10.40.11
        search:
          - healthclinic.local
```

**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`

### Step 3.3: Apply Network
```bash
# Apply configuration
sudo netplan apply

# Verify IP
ip addr show ens18

# Test connectivity
ping -c 3 10.10.40.1
ping -c 3 10.10.40.10
ping -c 3 8.8.8.8
```

---

## Part 4: Desktop GUI Installation

### Step 4.1: Update System
```bash
sudo apt update
sudo apt upgrade -y
```

### Step 4.2: Install Desktop
```bash
sudo apt install ubuntu-desktop-minimal -y
```

**Takes 10-15 minutes**

### Step 4.3: Configure Display Manager

**When prompted, select `gdm3`**

### Step 4.4: Reboot
```bash
sudo reboot
```

### Step 4.5: Login to Desktop

1. Click **webadmin**
2. Password: **Clinic@2025!**
3. Complete setup wizard (click Next/Skip through prompts)

### Step 4.6: Open Terminal

Press **Ctrl + Alt + T**

---

## Part 5: Web Server Setup

### Step 5.1: Install Tools
```bash
sudo apt update
sudo apt install -y curl wget git vim nano net-tools htop build-essential
```

### Step 5.2: Install Apache
```bash
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
sudo systemctl status apache2
```

Press **Q** to quit

### Step 5.3: Test Apache
```bash
curl http://localhost
```

**Or in browser:** `http://10.10.40.30`

### Step 5.4: Install Python
```bash
sudo apt install -y python3 python3-pip python3-venv python3-dev
sudo apt install -y libapache2-mod-wsgi-py3
sudo apt update
sudo apt install -y python3-venv python3-pip python3-flask
#Create a virtual environment:
python3 -m venv venv
#Activate it:
source venv/bin/activate
#Now install Flask-SQLAlchemy normally:
pip install Flask-SQLAlchemy
#No more errors because pip installs inside the venv, not system-wide.

# Verify
python3 --version
```

---

## Part 6: Database Configuration

### Step 6.1: Install PostgreSQL
```bash
sudo apt install -y postgresql postgresql-contrib python3-psycopg2
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo systemctl status postgresql
```

Press **Q**

### INSTALLING PSQL GUI
1. Install docker because ubuntu noble is not compatible to the repository of psql (compatibility issue)
```bash
sudo docker pull dpage/pgadmin4
sudo docker run -p 8080:80 -e PGADMIN_DEFAULT_EMAIL=admin@g3company.com -e PGADMIN_DEFAULT_PASSWORD=g3company\!@# -d dpage/pgadmin4
sudo docker ps
sudo docker restart "container id"
```bash

### THEN TEST IN BROWSER
http://localhost:8080
http://10.10.40.30:8080
EMAIL: admin@g3company.com
PW: g3company!@#

### Step 6.2: Create Database
```bash
# Switch to postgres user
sudo -i -u postgres
psql
```

**In PostgreSQL prompt:**
```sql
CREATE DATABASE healthclinic;
CREATE USER clinicadmin WITH PASSWORD 'Clinic@2025!';
GRANT ALL PRIVILEGES ON DATABASE healthclinic TO clinicadmin;
\c healthclinic
GRANT ALL ON SCHEMA public TO clinicadmin;
\l
\q
```
```bash
exit
```

### Step 6.3: Test Connection
```bash
psql -h localhost -U clinicadmin -d healthclinic -W
# Password: Clinic@2025!
```

Type `\q` to exit

---

## Part 7: Flask Application Setup

### Step 7.1: Create Directory
```bash
sudo mkdir -p /var/www/healthclinic
sudo mkdir -p /var/www/healthclinic/static/css
sudo mkdir -p /var/www/healthclinic/static/js
sudo mkdir -p /var/www/healthclinic/templates
sudo chown -R webadmin:www-data /var/www/healthclinic
sudo chmod -R 755 /var/www/healthclinic
```

### Step 7.2: Create Virtual Environment
```bash
cd /var/www/healthclinic
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install flask flask-sqlalchemy psycopg2-binary gunicorn
```

### Step 7.3: Create app.py
```bash
nano app.py
```

**Paste:**
```python
from flask import Flask, render_template, request, redirect, url_for, flash
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

app = Flask(__name__)
app.config['SECRET_KEY'] = 'healthclinic2025secret'
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://clinicadmin:Clinic@2025!@localhost/healthclinic'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

class Appointment(db.Model):
    __tablename__ = 'appointments'
    id = db.Column(db.Integer, primary_key=True)
    patient_name = db.Column(db.String(100), nullable=False)
    patient_email = db.Column(db.String(100), nullable=False)
    patient_phone = db.Column(db.String(20), nullable=False)
    appointment_date = db.Column(db.Date, nullable=False)
    appointment_time = db.Column(db.String(10), nullable=False)
    doctor = db.Column(db.String(50), nullable=False)
    reason = db.Column(db.Text, nullable=False)
    status = db.Column(db.String(20), default='Pending')
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

    def __repr__(self):
        return f'<Appointment {self.patient_name} - {self.appointment_date}>'

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/book', methods=['GET', 'POST'])
def book_appointment():
    if request.method == 'POST':
        patient_name = request.form.get('patient_name')
        patient_email = request.form.get('patient_email')
        patient_phone = request.form.get('patient_phone')
        appointment_date = datetime.strptime(request.form.get('appointment_date'), '%Y-%m-%d').date()
        appointment_time = request.form.get('appointment_time')
        doctor = request.form.get('doctor')
        reason = request.form.get('reason')
        
        new_appointment = Appointment(
            patient_name=patient_name,
            patient_email=patient_email,
            patient_phone=patient_phone,
            appointment_date=appointment_date,
            appointment_time=appointment_time,
            doctor=doctor,
            reason=reason
        )
        
        try:
            db.session.add(new_appointment)
            db.session.commit()
            flash('Appointment booked successfully!', 'success')
            return redirect(url_for('appointments'))
        except Exception as e:
            db.session.rollback()
            flash(f'Error booking appointment: {str(e)}', 'error')
            return redirect(url_for('book_appointment'))
    
    return render_template('book.html')

@app.route('/appointments')
def appointments():
    all_appointments = Appointment.query.order_by(Appointment.appointment_date.desc()).all()
    return render_template('appointments.html', appointments=all_appointments)

@app.route('/delete/<int:id>')
def delete_appointment(id):
    appointment = Appointment.query.get_or_404(id)
    try:
        db.session.delete(appointment)
        db.session.commit()
        flash('Appointment deleted successfully!', 'success')
    except Exception as e:
        flash(f'Error deleting appointment: {str(e)}', 'error')
    return redirect(url_for('appointments'))

with app.app_context():
    db.create_all()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`

### Step 7.4: Create base.html
```bash
nano templates/base.html
```

**Paste:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Healthcare Clinic{% endblock %}</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    <nav class="navbar">
        <div class="container">
            <h1>🏥 Healthcare Clinic</h1>
            <ul>
                <li><a href="{{ url_for('index') }}">Home</a></li>
                <li><a href="{{ url_for('book_appointment') }}">Book Appointment</a></li>
                <li><a href="{{ url_for('appointments') }}">View Appointments</a></li>
            </ul>
        </div>
    </nav>
    <div class="container content">
        {% with messages = get_flashed_messages(with_categories=true) %}
            {% if messages %}
                {% for category, message in messages %}
                    <div class="alert alert-{{ category }}">{{ message }}</div>
                {% endfor %}
            {% endif %}
        {% endwith %}
        {% block content %}{% endblock %}
    </div>
    <footer>
        <p>&copy; 2024 Healthcare Clinic | MITT Capstone Project</p>
    </footer>
</body>
</html>
```

**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`

### Step 7.5: Create index.html
```bash
nano templates/index.html
```

**Paste:**
```html
{% extends "base.html" %}
{% block title %}Home - Healthcare Clinic{% endblock %}
{% block content %}
<div class="hero">
    <h2>Welcome to Healthcare Clinic</h2>
    <p>Your health is our priority. Book an appointment today!</p>
    <a href="{{ url_for('book_appointment') }}" class="btn btn-primary">Book Appointment Now</a>
</div>
<div class="features">
    <div class="feature-box">
        <h3>👨‍⚕️ Expert Doctors</h3>
        <p>Our team of experienced healthcare professionals is here to help you.</p>
    </div>
    <div class="feature-box">
        <h3>📅 Easy Booking</h3>
        <p>Book your appointments online quickly and conveniently.</p>
    </div>
    <div class="feature-box">
        <h3>🏥 Quality Care</h3>
        <p>We provide comprehensive healthcare services for all your needs.</p>
    </div>
</div>
<div class="info-section">
    <h3>About Our System</h3>
    <p><strong>Server:</strong> WEB01 (10.10.40.30)</p>
    <p><strong>Stack:</strong> Flask + PostgreSQL + Apache</p>
    <p><strong>Project:</strong> MITT Network & Systems Administrator Capstone</p>
</div>
{% endblock %}
```

**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`

### Step 7.6: Create book.html
```bash
nano templates/book.html
```

**Paste:**
```html
{% extends "base.html" %}
{% block title %}Book Appointment{% endblock %}
{% block content %}
<h2>📅 Book an Appointment</h2>
<form method="POST" action="{{ url_for('book_appointment') }}" class="appointment-form">
    <div class="form-group">
        <label for="patient_name">Full Name *</label>
        <input type="text" id="patient_name" name="patient_name" required>
    </div>
    <div class="form-group">
        <label for="patient_email">Email *</label>
        <input type="email" id="patient_email" name="patient_email" required>
    </div>
    <div class="form-group">
        <label for="patient_phone">Phone Number *</label>
        <input type="tel" id="patient_phone" name="patient_phone" required>
    </div>
    <div class="form-row">
        <div class="form-group">
            <label for="appointment_date">Date *</label>
            <input type="date" id="appointment_date" name="appointment_date" required>
        </div>
        <div class="form-group">
            <label for="appointment_time">Time *</label>
            <select id="appointment_time" name="appointment_time" required>
                <option value="">Select time...</option>
                <option value="09:00 AM">09:00 AM</option>
                <option value="10:00 AM">10:00 AM</option>
                <option value="11:00 AM">11:00 AM</option>
                <option value="01:00 PM">01:00 PM</option>
                <option value="02:00 PM">02:00 PM</option>
                <option value="03:00 PM">03:00 PM</option>
                <option value="04:00 PM">04:00 PM</option>
            </select>
        </div>
    </div>
    <div class="form-group">
        <label for="doctor">Select Doctor *</label>
        <select id="doctor" name="doctor" required>
            <option value="">Choose a doctor...</option>
            <option value="Dr. Sarah Johnson">Dr. Sarah Johnson - General Practice</option>
            <option value="Dr. Michael Chen">Dr. Michael Chen - Pediatrics</option>
            <option value="Dr. Emily Rodriguez">Dr. Emily Rodriguez - Internal Medicine</option>
            <option value="Dr. David Kim">Dr. David Kim - Family Medicine</option>
        </select>
    </div>
    <div class="form-group">
        <label for="reason">Reason for Visit *</label>
        <textarea id="reason" name="reason" rows="4" required></textarea>
    </div>
    <button type="submit" class="btn btn-primary">Book Appointment</button>
    <a href="{{ url_for('index') }}" class="btn btn-secondary">Cancel</a>
</form>
{% endblock %}
```

**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`

### Step 7.7: Create appointments.html
```bash
nano templates/appointments.html
```

**Paste:**
```html
{% extends "base.html" %}
{% block title %}Appointments{% endblock %}
{% block content %}
<h2>📋 All Appointments</h2>
{% if appointments %}
<table class="appointments-table">
    <thead>
        <tr>
            <th>ID</th>
            <th>Patient Name</th>
            <th>Email</th>
            <th>Phone</th>
            <th>Date</th>
            <th>Time</th>
            <th>Doctor</th>
            <th>Status</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        {% for appointment in appointments %}
        <tr>
            <td>{{ appointment.id }}</td>
            <td>{{ appointment.patient_name }}</td>
            <td>{{ appointment.patient_email }}</td>
            <td>{{ appointment.patient_phone }}</td>
            <td>{{ appointment.appointment_date.strftime('%Y-%m-%d') }}</td>
            <td>{{ appointment.appointment_time }}</td>
            <td>{{ appointment.doctor }}</td>
            <td><span class="status-badge status-{{ appointment.status.lower() }}">{{ appointment.status }}</span></td>
            <td>
                <a href="{{ url_for('delete_appointment', id=appointment.id) }}" 
                   class="btn-delete" 
                   onclick="return confirm('Delete this appointment?')">Delete</a>
            </td>
        </tr>
        {% endfor %}
    </tbody>
</table>
{% else %}
<p class="no-data">No appointments found. <a href="{{ url_for('book_appointment') }}">Book your first appointment</a></p>
{% endif %}
{% endblock %}
```

**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`

### Step 7.8: Create style.css
```bash
nano static/css/style.css
```

**Paste:**
```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    line-height: 1.6;
    color: #333;
    background-color: #f4f4f4;
}

.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
}

.navbar {
    background-color: #2c3e50;
    color: white;
    padding: 1rem 0;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.navbar .container {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.navbar h1 {
    font-size: 1.5rem;
}

.navbar ul {
    display: flex;
    list-style: none;
    gap: 2rem;
}

.navbar a {
    color: white;
    text-decoration: none;
    transition: color 0.3s;
}

.navbar a:hover {
    color: #3498db;
}

.content {
    margin: 2rem auto;
    background: white;
    padding: 2rem;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    min-height: 60vh;
}

.hero {
    text-align: center;
    padding: 3rem 0;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border-radius: 8px;
    margin-bottom: 2rem;
}

.hero h2 {
    font-size: 2.5rem;
    margin-bottom: 1rem;
}

.hero p {
    font-size: 1.2rem;
    margin-bottom: 2rem;
}

.btn {
    display: inline-block;
    padding: 0.75rem 1.5rem;
    text-decoration: none;
    border-radius: 5px;
    transition: all 0.3s;
    cursor: pointer;
    border: none;
    font-size: 1rem;
}

.btn-primary {
    background-color: #3498db;
    color: white;
}

.btn-primary:hover {
    background-color: #2980b9;
}

.btn-secondary {
    background-color: #95a5a6;
    color: white;
}

.btn-secondary:hover {
    background-color: #7f8c8d;
}

.btn-delete {
    background-color: #e74c3c;
    color: white;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    text-decoration: none;
    font-size: 0.9rem;
}

.btn-delete:hover {
    background-color: #c0392b;
}

.features {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 2rem;
    margin: 2rem 0;
}

.feature-box {
    background: white;
    padding: 2rem;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    text-align: center;
    transition: transform 0.3s;
}

.feature-box:hover {
    transform: translateY(-5px);
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.feature-box h3 {
    margin-bottom: 1rem;
    color: #2c3e50;
}

.info-section {
    background: #ecf0f1;
    padding: 1.5rem;
    border-radius: 8px;
    margin-top: 2rem;
}

.info-section h3 {
    margin-bottom: 1rem;
    color: #2c3e50;
}

.appointment-form {
    max-width: 600px;
    margin: 2rem auto;
}

.form-group {
    margin-bottom: 1.5rem;
}

.form-row {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
}

.form-group label {
    display: block;
    margin-bottom: 0.5rem;
    font-weight: bold;
    color: #2c3e50;
}

.form-group input,
.form-group select,
.form-group textarea {
    width: 100%;
    padding: 0.75rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 1rem;
}

.form-group input:focus,
.form-group select:focus,
.form-group textarea:focus {
    outline: none;
    border-color: #3498db;
}

.appointments-table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 1rem;
}

.appointments-table th,
.appointments-table td {
    padding: 1rem;
    text-align: left;
    border-bottom: 1px solid #ddd;
}

.appointments-table th {
    background-color: #2c3e50;
    color: white;
}

.appointments-table tr:hover {
    background-color: #f5f5f5;
}

.status-badge {
    padding: 0.25rem 0.75rem;
    border-radius: 12px;
    font-size: 0.85rem;
    font-weight: bold;
}

.status-pending {
    background-color: #f39c12;
    color: white;
}

.status-confirmed {
    background-color: #27ae60;
    color: white;
}

.status-cancelled {
    background-color: #e74c3c;
    color: white;
}

.alert {
    padding: 1rem;
    margin-bottom: 1rem;
    border-radius: 4px;
}

.alert-success {
    background-color: #d4edda;
    color: #155724;
    border: 1px solid #c3e6cb;
}

.alert-error {
    background-color: #f8d7da;
    color: #721c24;
    border: 1px solid #f5c6cb;
}

.no-data {
    text-align: center;
    padding: 2rem;
    color: #7f8c8d;
}

footer {
    background-color: #2c3e50;
    color: white;
    text-align: center;
    padding: 1.5rem 0;
    margin-top: 2rem;
}

@media (max-width: 768px) {
    .navbar .container {
        flex-direction: column;
        gap: 1rem;
    }
    .navbar ul {
        flex-direction: column;
        gap: 0.5rem;
        text-align: center;
    }
    .form-row {
        grid-template-columns: 1fr;
    }
    .appointments-table {
        font-size: 0.85rem;
    }
    .appointments-table th,
    .appointments-table td {
        padding: 0.5rem;
    }
}
```

**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`

### Step 7.9: Test Flask
```bash
cd /var/www/healthclinic
source venv/bin/activate
python3 app.py
```

**Open browser:** `http://10.10.40.30:5000`

**Press Ctrl+C to stop**

---

## Part 8: Apache WSGI Configuration

### Step 8.1: Create wsgi.py
```bash
cd /var/www/healthclinic
nano wsgi.py
```

**Paste:**
```python
import sys
import os

sys.path.insert(0, '/var/www/healthclinic')

activate_this = '/var/www/healthclinic/venv/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

from app import app as application
```

**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`

### Step 8.2: Create Apache Config
```bash
sudo nano /etc/apache2/sites-available/healthclinic.conf
```

**Paste:**
```apache
<VirtualHost *:80>
    ServerName web01.healthclinic.local
    ServerAlias 10.10.40.30

    WSGIDaemonProcess healthclinic user=webadmin group=www-data threads=5 python-home=/var/www/healthclinic/venv
    WSGIScriptAlias / /var/www/healthclinic/wsgi.py

    <Directory /var/www/healthclinic>
        WSGIProcessGroup healthclinic
        WSGIApplicationGroup %{GLOBAL}
        Require all granted
    </Directory>

    Alias /static /var/www/healthclinic/static
    <Directory /var/www/healthclinic/static>
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/healthclinic_error.log
    CustomLog ${APACHE_LOG_DIR}/healthclinic_access.log combined
</VirtualHost>
```

**Save:** `Ctrl+O`, `Enter`, `Ctrl+X`

### Step 8.3: Enable Site
```bash
sudo a2dissite 000-default.conf
sudo a2ensite healthclinic.conf
sudo a2enmod wsgi
sudo apache2ctl configtest
sudo systemctl restart apache2
```

---

## Part 9: Testing & Verification

### Step 9.1: Access Application

**Open browser:** `http://10.10.40.30`

### Step 9.2: Book Test Appointment

1. Click "Book Appointment"
2. Fill form:
   - Name: Test Patient
   - Email: test@healthclinic.local
   - Phone: 204-555-0123
   - Date: Tomorrow
   - Time: 10:00 AM
   - Doctor: Dr. Sarah Johnson
   - Reason: Regular checkup
3. Click "Book Appointment"
4. Verify appears in appointments list

### Step 9.3: Verify Database
```bash
psql -h localhost -U clinicadmin -d healthclinic -W
SELECT * FROM appointments;
\q
```

---

## Part 10: Troubleshooting

### Cannot Access Website
```bash
sudo systemctl status apache2
sudo systemctl restart apache2
sudo ss -tulpn | grep :80
```

### Database Errors
```bash
sudo systemctl status postgresql
psql -h localhost -U clinicadmin -d healthclinic -W
```

### CSS Not Loading
```bash
ls -la /var/www/healthclinic/static/css/
sudo chown -R webadmin:www-data /var/www/healthclinic/static
sudo chmod -R 755 /var/www/healthclinic/static
sudo systemctl restart apache2
```

---

## Completion Checklist
```
✅ VM created and running
✅ Ubuntu 24.04 installed
✅ Static IP: 10.10.40.30/24
✅ Desktop GUI installed
✅ Apache running
✅ PostgreSQL running
✅ Database created
✅ Flask app deployed
✅ WSGI configured
✅ Website accessible
✅ Can book appointments
✅ Data saves to database
```

---

## Quick Reference

### Services
```bash
# Apache
sudo systemctl status apache2
sudo systemctl restart apache2

# PostgreSQL
sudo systemctl status postgresql

# View logs
sudo tail -f /var/log/apache2/healthclinic_error.log
```

### Database
```bash
# Connect
psql -h localhost -U clinicadmin -d healthclinic -W

# SQL commands
\l                  # List databases
\dt                 # List tables
SELECT * FROM appointments;
\q                  # Quit
```

### Network
```bash
ip addr show ens18
ping -c 3 10.10.40.1
ping -c 3 10.10.40.10
```

---

## File Structure
```
/var/www/healthclinic/
├── app.py
├── wsgi.py
├── venv/
├── static/
│   └── css/
│       └── style.css
└── templates/
    ├── base.html
    ├── index.html
    ├── book.html
    └── appointments.html
```

---

## Credits

**Project:** Healthcare Clinic IT Infrastructure  
**Institution:** M.I.T.T.  
**Student:** Paul Santos  
**Date:** December 2024

---

**END OF DOCUMENT - Copy everything from line 1 to here and paste to GitHub as WEB01_Complete_Setup_Guide.md**


# WEB01 - COMPLETE SETUP: LANDING PAGE + AUTHENTICATION SYSTEM
# Description: Automated setup for HealthClinic web application
# Includes: Enhanced UI, Database Models, Authentication, User Management
# Password for all users: g3company!@#

cd /var/www/healthclinic

# Activate virtual environment
source venv/bin/activate

# Install Flask-Login
pip install flask-login

# PART 1: CREATE ENHANCED LANDING PAGE

app/templates/index.html 

```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HealthClinic | Welcome</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
    <style>
        /* Custom hover effects */
        .card {
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }
        .card:hover {
            transform: translateY(-10px);
            box-shadow: 0 10px 20px rgba(0,0,0,0.15) !important;
        }
        .btn {
            transition: all 0.3s ease;
        }
        .btn:hover {
            transform: scale(1.05);
        }
        footer a {
            text-decoration: none;
            transition: opacity 0.3s ease;
        }
        footer a:hover {
            opacity: 0.8;
        }
    </style>
</head>
<body class="bg-light">

<!-- NAVBAR -->
<nav class="navbar navbar-expand-lg navbar-dark bg-primary">
    <div class="container">
        <a class="navbar-brand fw-bold" href="/">
            <i class="fas fa-hospital-alt"></i> HealthClinic
        </a>
        <div class="d-flex">
            <a href="/appointment" class="btn btn-light btn-sm me-2">
                <i class="fas fa-calendar-plus"></i> Book Appointment
            </a>
            <a href="/login" class="btn btn-outline-light btn-sm">
                <i class="fas fa-user-lock"></i> Staff Login
            </a>
        </div>
    </div>
</nav>

<!-- HERO SECTION -->
<header class="py-5 bg-white shadow-sm">
    <div class="container text-center">
        <h1 class="fw-bold display-4">Your Health, Our Priority.</h1>
        <p class="lead text-muted">Modern, reliable healthcare powered by technology.</p>
        <a href="/appointment" class="btn btn-primary btn-lg mt-3">
            <i class="fas fa-calendar-check"></i> Book an Appointment
        </a>
    </div>
</header>

<!-- SERVICES SECTION -->
<section class="py-5">
    <div class="container">
        <h2 class="text-center mb-5">Our Services</h2>
        <div class="row g-4">
            <div class="col-md-4">
                <div class="card shadow-sm p-4 text-center h-100">
                    <i class="fas fa-user-md fa-3x text-primary mb-3"></i>
                    <h5 class="fw-bold">General Checkups</h5>
                    <p class="text-muted">Routine health exams and consultations.</p>
                </div>
            </div>
            <div class="col-md-4">
                <div class="card shadow-sm p-4 text-center h-100">
                    <i class="fas fa-baby fa-3x text-primary mb-3"></i>
                    <h5 class="fw-bold">Pediatrics</h5>
                    <p class="text-muted">Medical care for infants, children, and teens.</p>
                </div>
            </div>
            <div class="col-md-4">
                <div class="card shadow-sm p-4 text-center h-100">
                    <i class="fas fa-syringe fa-3x text-primary mb-3"></i>
                    <h5 class="fw-bold">Laboratory Tests</h5>
                    <p class="text-muted">Complete diagnostics and lab test processing.</p>
                </div>
            </div>
        </div>
    </div>
</section>

<!-- WHY CHOOSE US SECTION -->
<section class="py-5 bg-light">
    <div class="container text-center">
        <h2 class="mb-5">Why Choose HealthClinic?</h2>
        <div class="row stats-section">
            <div class="col-md-3 mb-4">
                <i class="fas fa-user-md fa-2x text-primary mb-2"></i>
                <h2 class="text-primary fw-bold">4</h2>
                <p>Expert Doctors</p>
            </div>
            <div class="col-md-3 mb-4">
                <i class="fas fa-users fa-2x text-primary mb-2"></i>
                <h2 class="text-primary fw-bold">1000+</h2>
                <p>Patients Served</p>
            </div>
            <div class="col-md-3 mb-4">
                <i class="fas fa-clock fa-2x text-primary mb-2"></i>
                <h2 class="text-primary fw-bold">24/7</h2>
                <p>Emergency Support</p>
            </div>
            <div class="col-md-3 mb-4">
                <i class="fas fa-heart fa-2x text-primary mb-2"></i>
                <h2 class="text-primary fw-bold">100%</h2>
                <p>Patient Satisfaction</p>
            </div>
        </div>
    </div>
</section>

<!-- CONTACT/HOURS SECTION -->
<section class="py-5 bg-white">
    <div class="container">
        <div class="row">
            <div class="col-md-6 mb-4">
                <h4><i class="fas fa-map-marker-alt text-primary"></i> Location</h4>
                <p class="ms-4">
                    123 Main Street<br>
                    Winnipeg, MB R3C 1A5<br>
                    Canada
                </p>
            </div>
            <div class="col-md-6 mb-4">
                <h4><i class="fas fa-clock text-primary"></i> Clinic Hours</h4>
                <p class="ms-4">
                    Monday - Friday: 8:00 AM - 6:00 PM<br>
                    Saturday: 9:00 AM - 2:00 PM<br>
                    Sunday: Closed
                </p>
            </div>
        </div>
    </div>
</section>

<!-- FOOTER -->
<footer class="bg-primary text-white py-4">
    <div class="container">
        <div class="row">
            <div class="col-md-4 mb-3">
                <h5><i class="fas fa-hospital-alt"></i> HealthClinic</h5>
                <p>Your trusted healthcare partner</p>
            </div>
            <div class="col-md-4 mb-3">
                <h6>Quick Links</h6>
                <ul class="list-unstyled">
                    <li><a href="/appointment" class="text-white"><i class="fas fa-calendar"></i> Book Appointment</a></li>
                    <li><a href="/login" class="text-white"><i class="fas fa-sign-in-alt"></i> Staff Login</a></li>
                    <li><a href="#" class="text-white"><i class="fas fa-info-circle"></i> About Us</a></li>
                </ul>
            </div>
            <div class="col-md-4 mb-3">
                <h6>Contact</h6>
                <p>
                    <i class="fas fa-phone"></i> (204) 555-0123<br>
                    <i class="fas fa-envelope"></i> info@healthclinic.local
                </p>
            </div>
        </div>
        <hr class="bg-white">
        <p class="text-center mb-0">
            © 2025 HealthClinic • MITT Capstone Project • All Rights Reserved
        </p>
    </div>
</footer>

</body>
</html>
```

# PART 2: CREATE DATABASE MODELS

app/models.py 

```bash
from . import db
from werkzeug.security import generate_password_hash, check_password_hash
from flask_login import UserMixin
from datetime import datetime

class User(UserMixin, db.Model):
    __tablename__ = 'users'

    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(255), nullable=False)
    full_name = db.Column(db.String(100), nullable=False)
    role = db.Column(db.String(20), nullable=False)  # doctor, nurse, admin, it
    phone = db.Column(db.String(20))
    is_active = db.Column(db.Boolean, default=True)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    last_login = db.Column(db.DateTime)

    def set_password(self, password):
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        return check_password_hash(self.password_hash, password)

    def __repr__(self):
        return f'<User {self.username} - {self.role}>'

class Patient(db.Model):
    __tablename__ = 'patients'

    id = db.Column(db.Integer, primary_key=True)
    patient_id = db.Column(db.String(20), unique=True)
    first_name = db.Column(db.String(50), nullable=False)
    last_name = db.Column(db.String(50), nullable=False)
    date_of_birth = db.Column(db.Date)
    gender = db.Column(db.String(10))
    phone = db.Column(db.String(20))
    email = db.Column(db.String(120))
    address = db.Column(db.Text)
    emergency_contact = db.Column(db.String(100))
    emergency_phone = db.Column(db.String(20))
    blood_type = db.Column(db.String(5))
    allergies = db.Column(db.Text)
    medical_notes = db.Column(db.Text)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    # Relationship
    appointments = db.relationship('Appointment', backref='patient', lazy=True)

    def __repr__(self):
        return f'<Patient {self.first_name} {self.last_name}>'

class Appointment(db.Model):
    __tablename__ = 'appointments'

    id = db.Column(db.Integer, primary_key=True)
    patient_id = db.Column(db.Integer, db.ForeignKey('patients.id'), nullable=False)
    patient_name = db.Column(db.String(100), nullable=False)
    patient_email = db.Column(db.String(120))
    patient_phone = db.Column(db.String(20))
    appointment_date = db.Column(db.Date, nullable=False)
    appointment_time = db.Column(db.String(10), nullable=False)
    doctor = db.Column(db.String(50), nullable=False)
    department = db.Column(db.String(50))
    reason = db.Column(db.Text, nullable=False)
    status = db.Column(db.String(20), default='pending')  # pending, confirmed, cancelled, completed
    notes = db.Column(db.Text)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    def __repr__(self):
        return f'<Appointment {self.patient_name} - {self.appointment_date}>'
```

# PART 3: CONFIGURE FLASK AUTHENTICATION

app/__init__.py

```bash
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager

db = SQLAlchemy()
login_manager = LoginManager()

def create_app():
    app = Flask(__name__)

    # Secret key for sessions
    app.config['SECRET_KEY'] = 'healthclinic2025-secret-key-change-in-production'

    # PostgreSQL connection
    app.config['SQLALCHEMY_DATABASE_URI'] = (
        'postgresql://webadmin:T%40ylorSwift13@127.0.0.1:5432/healthclinic'
    )
    app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

    # Initialize extensions
    db.init_app(app)
    login_manager.init_app(app)
    login_manager.login_view = 'main.login'
    login_manager.login_message = 'Please log in to access this page.'

    # User loader for Flask-Login
    @login_manager.user_loader
    def load_user(user_id):
        from .models import User
        return User.query.get(int(user_id))

    # Register blueprints
    from .routes import main_bp
    app.register_blueprint(main_bp)

    return app
    ```

# PART 4: CREATE LOGIN PAGE

app/templates/login.html

```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Staff Login | HealthClinic</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
    <style>
        body {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .login-card {
            background: white;
            border-radius: 15px;
            box-shadow: 0 10px 40px rgba(0,0,0,0.2);
            overflow: hidden;
            max-width: 900px;
            width: 100%;
        }
        .login-left {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 3rem;
            display: flex;
            flex-direction: column;
            justify-content: center;
        }
        .login-right {
            padding: 3rem;
        }
        .form-control:focus {
            border-color: #667eea;
            box-shadow: 0 0 0 0.2rem rgba(102, 126, 234, 0.25);
        }
        .btn-login {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            border: none;
            padding: 0.75rem;
            font-weight: bold;
        }
        .btn-login:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
        }
        .role-badge {
            display: inline-block;
            padding: 0.5rem 1rem;
            margin: 0.25rem;
            border-radius: 20px;
            background: rgba(255,255,255,0.2);
            font-size: 0.9rem;
        }
    </style>
</head>
<body>

<div class="container">
    <div class="login-card row g-0">
        <!-- LEFT SIDE -->
        <div class="col-md-5 login-left">
            <div class="text-center mb-4">
                <i class="fas fa-hospital-alt fa-4x mb-3"></i>
                <h2 class="fw-bold">HealthClinic</h2>
                <p class="mb-0">Staff Portal</p>
            </div>
            <hr class="bg-white mb-4">
            <h5 class="mb-3">Authorized Personnel Only</h5>
            <div class="mb-3">
                <span class="role-badge"><i class="fas fa-user-md"></i> Doctors</span>
                <span class="role-badge"><i class="fas fa-user-nurse"></i> Nurses</span>
            </div>
            <div>
                <span class="role-badge"><i class="fas fa-user-shield"></i> Administrators</span>
                <span class="role-badge"><i class="fas fa-laptop-code"></i> IT Staff</span>
            </div>
        </div>

        <!-- RIGHT SIDE - LOGIN FORM -->
        <div class="col-md-7 login-right">
            <div class="mb-4">
                <h3 class="fw-bold mb-2">Welcome Back</h3>
                <p class="text-muted">Sign in to access your dashboard</p>
            </div>

            <!-- Flash Messages -->
            {% with messages = get_flashed_messages(with_categories=true) %}
                {% if messages %}
                    {% for category, message in messages %}
                        <div class="alert alert-{{ 'danger' if category == 'error' else category }} alert-dismissible fade show" role="alert">
                            {{ message }}
                            <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                        </div>
                    {% endfor %}
                {% endif %}
            {% endwith %}

            <!-- Login Form -->
            <form method="POST" action="{{ url_for('main.login') }}">
                <div class="mb-3">
                    <label class="form-label">
                        <i class="fas fa-user"></i> Username
                    </label>
                    <input type="text" name="username" class="form-control form-control-lg"
                           placeholder="Enter your username" required autofocus>
                </div>

                <div class="mb-3">
                    <label class="form-label">
                        <i class="fas fa-lock"></i> Password
                    </label>
                    <input type="password" name="password" class="form-control form-control-lg"
                           placeholder="Enter your password" required>
                </div>

                <div class="mb-3 form-check">
                    <input type="checkbox" name="remember" class="form-check-input" id="remember">
                    <label class="form-check-label" for="remember">Remember me</label>
                </div>

                <button type="submit" class="btn btn-primary btn-login w-100 btn-lg">
                    <i class="fas fa-sign-in-alt"></i> Sign In
                </button>
            </form>

            <hr class="my-4">

            <div class="text-center">
                <a href="/" class="text-decoration-none">
                    <i class="fas fa-arrow-left"></i> Back to Homepage
                </a>
            </div>

            <div class="text-center mt-3">
                <small class="text-muted">
                    <i class="fas fa-info-circle"></i> Having trouble logging in? Contact IT Support
                </small>
            </div>
        </div>
    </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

# PART 5: UPDATE ROUTES WITH AUTHENTICATION

app/routes.py 

```bash
from flask import Blueprint, render_template, request, redirect, url_for, flash
from flask_login import login_user, logout_user, login_required, current_user
from datetime import datetime, date
from . import db
from .models import User, Patient, Appointment

main_bp = Blueprint('main', __name__)

# ========== PUBLIC ROUTES ==========

@main_bp.route('/')
def index():
    return render_template('index.html')

@main_bp.route('/appointment')
def appointment():
    return render_template('appointment.html')

# ========== AUTHENTICATION ROUTES ==========

@main_bp.route('/login', methods=['GET', 'POST'])
def login():
    if current_user.is_authenticated:
        return redirect(url_for('main.dashboard'))
    
    if request.method == 'POST':
        username = request.form.get('username')
        password = request.form.get('password')
        remember = True if request.form.get('remember') else False
        
        user = User.query.filter_by(username=username).first()
        
        if user and user.check_password(password):
            if user.is_active:
                login_user(user, remember=remember)
                user.last_login = datetime.utcnow()
                db.session.commit()
                flash(f'Welcome back, {user.full_name}!', 'success')
                return redirect(url_for('main.dashboard'))
            else:
                flash('Your account has been deactivated. Contact admin.', 'error')
        else:
            flash('Invalid username or password.', 'error')
    
    return render_template('login.html')

@main_bp.route('/logout')
@login_required
def logout():
    logout_user()
    flash('You have been logged out successfully.', 'info')
    return redirect(url_for('main.index'))

# ========== DASHBOARD (Protected) ==========

@main_bp.route('/dashboard')
@login_required
def dashboard():
    try:
        # Get statistics with safe defaults
        total_patients = Patient.query.count() or 0
        total_appointments = Appointment.query.count() or 0
        pending_appointments = Appointment.query.filter_by(status='pending').count() or 0
        
        # Today's appointments (using date.today() instead of datetime.utcnow().date())
        today = date.today()
        today_appointments = Appointment.query.filter_by(appointment_date=today).count() or 0
        
        # Get recent appointments
        recent_appointments = Appointment.query.order_by(Appointment.created_at.desc()).limit(5).all()
        
    except Exception as e:
        # If any error, use safe defaults
        print(f"Dashboard error: {e}")
        total_patients = 0
        total_appointments = 0
        pending_appointments = 0
        today_appointments = 0
        recent_appointments = []
    
    return render_template('dashboard.html',
                         total_patients=total_patients,
                         total_appointments=total_appointments,
                         pending_appointments=pending_appointments,
                         today_appointments=today_appointments,
                         recent_appointments=recent_appointments)
```

---
# PART 6: CREATE USER MANAGEMENT SCRIPT
---

cat > app/create_user.py << 'EOF'
#!/usr/bin/env python3
import sys, os

current_dir = os.path.dirname(os.path.abspath(__file__))
parent_dir = os.path.dirname(current_dir)
sys.path.insert(0, parent_dir)

from app import create_app, db
from app.models import User

DEFAULT_PASSWORD = 'g3company!@#'

def create_user(username, email, full_name, role, phone=None):
    app = create_app()
    with app.app_context():
        existing = User.query.filter_by(username=username).first()
        if existing:
            print(f"⚠️  User '{username}' already exists!")
            return False

        user = User(username=username, email=email, full_name=full_name, role=role, phone=phone, is_active=True)
        user.set_password(DEFAULT_PASSWORD)
        db.session.add(user)
        db.session.commit()

        print(f"✅ Created: {username:20} | {role:10} | {full_name}")
        return True

def list_users():
    app = create_app()
    with app.app_context():
        users = User.query.all()
        print("\n...
EOF

# ============================================================================

##  Patients Management page
Patients Management Page
What We're Building:

1. Patients List - Table with all patients
2. Add New Patient - Form to register new patients
3. View Patient Details - Full patient information
4. Edit Patient - Update patient info
5. Delete Patient - Remove patient (with confirmation)
6. Search/Filter - Find patients by name or ID

# STEP 1: RECREATE TABLES WITH CORRECT SCHEMA

echo ""
echo "Creating tables with correct schema..."

python3 << 'PYEOF'
from app import create_app, db
from app.models import User, Patient, Appointment

app = create_app()

with app.app_context():
    print("✅ Creating all tables...")
    db.create_all()
    
    print("\n✅ Tables created successfully!")
    print("\nVerifying schema...")
    
    # Verify tables
    from sqlalchemy import inspect
    inspector = inspect(db.engine)
    
    for table in ['users', 'patients', 'appointments']:
        print(f"\n📋 {table.upper()} TABLE:")
        columns = inspector.get_columns(table)
        for col in columns:
            pk = " (PRIMARY KEY)" if col.get('primary_key') else ""
            print(f"  • {col['name']:20} {str(col['type']):20} {pk}")
PYEOF

# STEP 2: RECREATE USERS

echo ""
echo "Creating default users..."

python3 app/create_user.py

# STEP 3: VERIFY NEW SCHEMA

echo ""
echo "Verifying new patients table schema..."

psql -h localhost -U webadmin -d healthclinic << 'SQLEOF'
-- Show new patients table structure
\d patients

-- Should now show:
-- id (PRIMARY KEY)
-- patient_id (VARCHAR)
-- first_name (VARCHAR)
-- last_name (VARCHAR)
-- etc.

\q
SQLEOF

# STEP 5: RESTART APACHE

sudo systemctl restart apache2

# ============================================================================

## Appointments Management
What We're Building:

Appointments List - View all appointments with filters
Book Appointment - Create new appointment (linked to patients)
View Appointment - See full details
Manage Status - Approve/Reject/Cancel/Complete appointments
Calendar View - Visual calendar display (optional enhancement)

# STEP 1: UPDATE ROUTES.PY (ADD APPOINTMENTS ROUTES)

 ```bash
from flask import Blueprint, render_template, request, redirect, url_for, flash
from flask_login import login_user, logout_user, login_required, current_user
from datetime import datetime, date
from . import db
from .models import User, Patient, Appointment

main_bp = Blueprint('main', __name__)

# ========== PUBLIC ROUTES ==========

@main_bp.route('/')
def index():
    return render_template('index.html')

@main_bp.route('/appointment')
def appointment():
    return render_template('appointment.html')

# ========== AUTHENTICATION ==========

@main_bp.route('/login', methods=['GET', 'POST'])
def login():
    if current_user.is_authenticated:
        return redirect(url_for('main.dashboard'))
    if request.method == 'POST':
        username = request.form.get('username')
        password = request.form.get('password')
        remember = True if request.form.get('remember') else False
        user = User.query.filter_by(username=username).first()
        if user and user.check_password(password):
            if user.is_active:
                login_user(user, remember=remember)
                user.last_login = datetime.utcnow()
                db.session.commit()
                flash(f'Welcome back, {user.full_name}!', 'success')
                return redirect(url_for('main.dashboard'))
            else:
                flash('Your account has been deactivated. Contact admin.', 'error')
        else:
            flash('Invalid username or password.', 'error')
    return render_template('login.html')

@main_bp.route('/logout')
@login_required
def logout():
    logout_user()
    flash('You have been logged out successfully.', 'info')
    return redirect(url_for('main.index'))

# ========== DASHBOARD ==========

@main_bp.route('/dashboard')
@login_required
def dashboard():
    try:
        total_patients = Patient.query.count() or 0
        total_appointments = Appointment.query.count() or 0
        pending_appointments = Appointment.query.filter_by(status='pending').count() or 0
        today = date.today()
        today_appointments = Appointment.query.filter_by(appointment_date=today).count() or 0
        recent_appointments = Appointment.query.order_by(Appointment.created_at.desc()).limit(5).all()
    except Exception as e:
        print(f"Dashboard error: {e}")
        total_patients = 0
        total_appointments = 0
        pending_appointments = 0
        today_appointments = 0
        recent_appointments = []
    return render_template('dashboard.html',
                         total_patients=total_patients,
                         total_appointments=total_appointments,
                         pending_appointments=pending_appointments,
                         today_appointments=today_appointments,
                         recent_appointments=recent_appointments)

# ========== PATIENTS ==========

@main_bp.route('/patients')
@login_required
def patients_list():
    try:
        search = request.args.get('search', '')
        if search:
            patients = Patient.query.filter(
                (Patient.first_name.ilike(f'%{search}%')) |
                (Patient.last_name.ilike(f'%{search}%')) |
                (Patient.patient_id.ilike(f'%{search}%'))
            ).order_by(Patient.created_at.desc()).all()
        else:
            patients = Patient.query.order_by(Patient.created_at.desc()).all()
        return render_template('patients_list.html', patients=patients, search=search)
    except Exception as e:
        flash(f'Error loading patients: {str(e)}', 'error')
        return render_template('patients_list.html', patients=[], search='')

@main_bp.route('/patients/add', methods=['GET', 'POST'])
@login_required
def patients_add():
    if request.method == 'POST':
        try:
            last_patient = Patient.query.order_by(Patient.id.desc()).first()
            if last_patient and last_patient.patient_id:
                last_num = int(last_patient.patient_id.replace('P', ''))
                patient_id = f'P{last_num + 1:05d}'
            else:
                patient_id = 'P00001'
            dob_str = request.form.get('date_of_birth')
            dob = datetime.strptime(dob_str, '%Y-%m-%d').date() if dob_str else None
            patient = Patient(
                patient_id=patient_id,
                first_name=request.form.get('first_name'),
                last_name=request.form.get('last_name'),
                date_of_birth=dob,
                gender=request.form.get('gender'),
                phone=request.form.get('phone'),
                email=request.form.get('email'),
                address=request.form.get('address'),
                emergency_contact=request.form.get('emergency_contact'),
                emergency_phone=request.form.get('emergency_phone'),
                blood_type=request.form.get('blood_type'),
                allergies=request.form.get('allergies'),
                medical_notes=request.form.get('medical_notes')
            )
            db.session.add(patient)
            db.session.commit()
            flash(f'Patient {patient.first_name} {patient.last_name} added successfully! (ID: {patient_id})', 'success')
            return redirect(url_for('main.patients_list'))
        except Exception as e:
            db.session.rollback()
            flash(f'Error adding patient: {str(e)}', 'error')
    return render_template('patients_add.html')

@main_bp.route('/patients/view/<int:id>')
@login_required
def patients_view(id):
    try:
        patient = Patient.query.get_or_404(id)
        appointments = Appointment.query.filter_by(patient_id=id).order_by(Appointment.appointment_date.desc()).all()
        return render_template('patients_view.html', patient=patient, appointments=appointments)
    except Exception as e:
        flash(f'Error: {str(e)}', 'error')
        return redirect(url_for('main.patients_list'))

@main_bp.route('/patients/edit/<int:id>', methods=['GET', 'POST'])
@login_required
def patients_edit(id):
    try:
        patient = Patient.query.get_or_404(id)
        if request.method == 'POST':
            dob_str = request.form.get('date_of_birth')
            patient.first_name = request.form.get('first_name')
            patient.last_name = request.form.get('last_name')
            patient.date_of_birth = datetime.strptime(dob_str, '%Y-%m-%d').date() if dob_str else None
            patient.gender = request.form.get('gender')
            patient.phone = request.form.get('phone')
            patient.email = request.form.get('email')
            patient.address = request.form.get('address')
            patient.emergency_contact = request.form.get('emergency_contact')
            patient.emergency_phone = request.form.get('emergency_phone')
            patient.blood_type = request.form.get('blood_type')
            patient.allergies = request.form.get('allergies')
            patient.medical_notes = request.form.get('medical_notes')
            patient.updated_at = datetime.utcnow()
            db.session.commit()
            flash(f'Patient {patient.first_name} {patient.last_name} updated successfully!', 'success')
            return redirect(url_for('main.patients_view', id=id))
        return render_template('patients_edit.html', patient=patient)
    except Exception as e:
        flash(f'Error: {str(e)}', 'error')
        return redirect(url_for('main.patients_list'))

@main_bp.route('/patients/delete/<int:id>', methods=['POST'])
@login_required
def patients_delete(id):
    try:
        patient = Patient.query.get_or_404(id)
        name = f"{patient.first_name} {patient.last_name}"
        db.session.delete(patient)
        db.session.commit()
        flash(f'Patient {name} deleted successfully!', 'success')
    except Exception as e:
        db.session.rollback()
        flash(f'Error deleting patient: {str(e)}', 'error')
    return redirect(url_for('main.patients_list'))

# ========== APPOINTMENTS ==========

@main_bp.route('/appointments')
@login_required
def appointments_list():
    try:
        status_filter = request.args.get('status', '')
        date_filter = request.args.get('date', '')
        query = Appointment.query
        if status_filter:
            query = query.filter_by(status=status_filter)
        if date_filter:
            try:
                filter_date = datetime.strptime(date_filter, '%Y-%m-%d').date()
                query = query.filter_by(appointment_date=filter_date)
            except:
                pass
        appointments = query.order_by(Appointment.appointment_date.desc(), Appointment.appointment_time.desc()).all()
        return render_template('appointments_list.html', 
                             appointments=appointments,
                             status_filter=status_filter,
                             date_filter=date_filter)
    except Exception as e:
        flash(f'Error loading appointments: {str(e)}', 'error')
        return render_template('appointments_list.html', appointments=[], status_filter='', date_filter='')

@main_bp.route('/appointments/book', methods=['GET', 'POST'])
@login_required
def appointments_book():
    if request.method == 'POST':
        try:
            patient_id = request.form.get('patient_id')
            patient = Patient.query.get(patient_id)
            
            if not patient:
                flash('Patient not found!', 'error')
                return redirect(url_for('main.appointments_book'))
            
            appt_date = datetime.strptime(request.form.get('appointment_date'), '%Y-%m-%d').date()
            
            appointment = Appointment(
                patient_id=patient.id,
                patient_name=f"{patient.first_name} {patient.last_name}",
                patient_email=patient.email,
                patient_phone=patient.phone,
                appointment_date=appt_date,
                appointment_time=request.form.get('appointment_time'),
                doctor=request.form.get('doctor'),
                department=request.form.get('department'),
                reason=request.form.get('reason'),
                status='confirmed',
                notes=request.form.get('notes')
            )
            
            db.session.add(appointment)
            db.session.commit()
            
            flash(f'Appointment booked successfully for {patient.first_name} {patient.last_name} on {appt_date}!', 'success')
            return redirect(url_for('main.appointments_list'))
            
        except Exception as e:
            db.session.rollback()
            flash(f'Error booking appointment: {str(e)}', 'error')
            return redirect(url_for('main.appointments_book'))
    
    try:
        patients = Patient.query.order_by(Patient.first_name).all()
        return render_template('appointments_book.html', patients=patients)
    except Exception as e:
        flash(f'Error loading patients: {str(e)}', 'error')
        return redirect(url_for('main.dashboard'))

@main_bp.route('/appointments/view/<int:id>')
@login_required
def appointments_view(id):
    try:
        appointment = Appointment.query.get_or_404(id)
        patient = Patient.query.get(appointment.patient_id)
        return render_template('appointments_view.html', appointment=appointment, patient=patient)
    except Exception as e:
        flash(f'Error: {str(e)}', 'error')
        return redirect(url_for('main.appointments_list'))

@main_bp.route('/appointments/update-status/<int:id>', methods=['POST'])
@login_required
def appointments_update_status(id):
    try:
        appointment = Appointment.query.get_or_404(id)
        new_status = request.form.get('status')
        appointment.status = new_status
        appointment.updated_at = datetime.utcnow()
        db.session.commit()
        flash(f'Appointment status updated to {new_status}!', 'success')
    except Exception as e:
        db.session.rollback()
        flash(f'Error updating status: {str(e)}', 'error')
    return redirect(url_for('main.appointments_view', id=id))

@main_bp.route('/appointments/delete/<int:id>', methods=['POST'])
@login_required
def appointments_delete(id):
    try:
        appointment = Appointment.query.get_or_404(id)
        db.session.delete(appointment)
        db.session.commit()
        flash('Appointment deleted successfully!', 'success')
    except Exception as e:
        db.session.rollback()
        flash(f'Error deleting appointment: {str(e)}', 'error')
    return redirect(url_for('main.appointments_list'))
```

#STEP 2: CREATE APPOINTMENTS LIST PAGE

app/templates/appointments_list.html

```bash
{% extends "base_dashboard.html" %}
{% block title %}Appointments | HealthClinic{% endblock %}
{% block content %}
<div class="d-flex justify-content-between align-items-center mb-4">
    <h2><i class="fas fa-calendar-alt"></i> Appointments</h2>
    <a href="/appointments/book" class="btn btn-primary"><i class="fas fa-calendar-plus"></i> Book Appointment</a>
</div>

<!-- FILTERS -->
<div class="card mb-4">
    <div class="card-body">
        <form method="GET" action="/appointments" class="row g-3">
            <div class="col-md-4">
                <label class="form-label">Status</label>
                <select name="status" class="form-control">
                    <option value="">All Status</option>
                    <option value="pending" {% if status_filter == 'pending' %}selected{% endif %}>Pending</option>
                    <option value="confirmed" {% if status_filter == 'confirmed' %}selected{% endif %}>Confirmed</option>
                    <option value="completed" {% if status_filter == 'completed' %}selected{% endif %}>Completed</option>
                    <option value="cancelled" {% if status_filter == 'cancelled' %}selected{% endif %}>Cancelled</option>
                </select>
            </div>
            <div class="col-md-4">
                <label class="form-label">Date</label>
                <input type="date" name="date" class="form-control" value="{{ date_filter }}">
            </div>
            <div class="col-md-4">
                <label class="form-label">&nbsp;</label>
                <button type="submit" class="btn btn-primary w-100"><i class="fas fa-filter"></i> Filter</button>
            </div>
        </form>
    </div>
</div>

<!-- APPOINTMENTS TABLE -->
{% if appointments %}
<div class="card">
    <div class="card-body">
        <div class="table-responsive">
            <table class="table table-hover">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Patient</th>
                        <th>Doctor</th>
                        <th>Date</th>
                        <th>Time</th>
                        <th>Department</th>
                        <th>Status</th>
                        <th>Actions</th>
                    </tr>
                </thead>
                <tbody>
                    {% for appt in appointments %}
                    <tr>
                        <td><strong>#{{ appt.id }}</strong></td>
                        <td><i class="fas fa-user"></i> {{ appt.patient_name }}</td>
                        <td><i class="fas fa-user-md"></i> {{ appt.doctor }}</td>
                        <td><i class="fas fa-calendar"></i> {{ appt.appointment_date.strftime('%Y-%m-%d') }}</td>
                        <td><i class="fas fa-clock"></i> {{ appt.appointment_time }}</td>
                        <td>{{ appt.department or 'N/A' }}</td>
                        <td>
                            <span class="badge badge-{{ appt.status }}">{{ appt.status|capitalize }}</span>
                        </td>
                        <td>
                            <a href="/appointments/view/{{ appt.id }}" class="btn btn-sm btn-info"><i class="fas fa-eye"></i></a>
                            <form method="POST" action="/appointments/delete/{{ appt.id }}" style="display:inline;" onsubmit="return confirm('Delete this appointment?');">
                                <button type="submit" class="btn btn-sm btn-danger"><i class="fas fa-trash"></i></button>
                            </form>
                        </td>
                    </tr>
                    {% endfor %}
                </tbody>
            </table>
        </div>
        <p class="text-muted mt-3">Total: {{ appointments|length }} appointment(s)</p>
    </div>
</div>
{% else %}
<div class="card">
    <div class="card-body text-center py-5">
        <i class="fas fa-calendar-times fa-3x text-muted mb-3"></i>
        <p class="text-muted">No appointments found.</p>
        <a href="/appointments/book" class="btn btn-primary">Book First Appointment</a>
    </div>
</div>
{% endif %}
{% endblock %}

# ============================================================================
# STEP 3: CREATE BOOK APPOINTMENT PAGE
# ============================================================================

app/templates/appointments_book.html

```bash
{% extends "base_dashboard.html" %}
{% block title %}Book Appointment{% endblock %}
{% block content %}
<div class="d-flex justify-content-between align-items-center mb-4">
    <h2><i class="fas fa-calendar-plus"></i> Book New Appointment</h2>
    <a href="/appointments" class="btn btn-secondary"><i class="fas fa-arrow-left"></i> Back</a>
</div>

<div class="card">
    <div class="card-body">
        <form method="POST" action="/appointments/book">
            <h5 class="mb-3">Patient Information</h5>
            <div class="mb-3">
                <label class="form-label">Select Patient *</label>
                <select name="patient_id" class="form-control" required>
                    <option value="">Choose patient...</option>
                    {% for patient in patients %}
                    <option value="{{ patient.id }}">{{ patient.patient_id }} - {{ patient.first_name }} {{ patient.last_name }}</option>
                    {% endfor %}
                </select>
                <small class="text-muted">Don't see the patient? <a href="/patients/add" target="_blank">Add new patient</a></small>
            </div>

            <h5 class="mb-3 mt-4">Appointment Details</h5>
            <div class="row">
                <div class="col-md-6 mb-3">
                    <label class="form-label">Date *</label>
                    <input type="date" name="appointment_date" class="form-control" id="appt_date" required>
                </div>
                <div class="col-md-6 mb-3">
                    <label class="form-label">Time *</label>
                    <select name="appointment_time" class="form-control" required>
                        <option value="">Select time...</option>
                        <option value="08:00 AM">08:00 AM</option>
                        <option value="09:00 AM">09:00 AM</option>
                        <option value="10:00 AM">10:00 AM</option>
                        <option value="11:00 AM">11:00 AM</option>
                        <option value="01:00 PM">01:00 PM</option>
                        <option value="02:00 PM">02:00 PM</option>
                        <option value="03:00 PM">03:00 PM</option>
                        <option value="04:00 PM">04:00 PM</option>
                        <option value="05:00 PM">05:00 PM</option>
                    </select>
                </div>
            </div>

            <div class="row">
                <div class="col-md-6 mb-3">
                    <label class="form-label">Doctor *</label>
                    <select name="doctor" class="form-control" required>
                        <option value="">Select doctor...</option>
                        <option value="Dr. Sarah Johnson">Dr. Sarah Johnson - General Practice</option>
                        <option value="Dr. Michael Chen">Dr. Michael Chen - Pediatrics</option>
                        <option value="Dr. Emily Rodriguez">Dr. Emily Rodriguez - Internal Medicine</option>
                        <option value="Dr. David Kim">Dr. David Kim - Family Medicine</option>
                    </select>
                </div>
                <div class="col-md-6 mb-3">
                    <label class="form-label">Department</label>
                    <select name="department" class="form-control">
                        <option value="">Select department...</option>
                        <option value="General Practice">General Practice</option>
                        <option value="Pediatrics">Pediatrics</option>
                        <option value="Internal Medicine">Internal Medicine</option>
                        <option value="Family Medicine">Family Medicine</option>
                        <option value="Emergency">Emergency</option>
                    </select>
                </div>
            </div>

            <div class="mb-3">
                <label class="form-label">Reason for Visit *</label>
                <textarea name="reason" class="form-control" rows="3" required placeholder="Describe the reason for the appointment..."></textarea>
            </div>

            <div class="mb-3">
                <label class="form-label">Notes (Optional)</label>
                <textarea name="notes" class="form-control" rows="2" placeholder="Any additional notes..."></textarea>
            </div>

            <div class="mt-4">
                <button type="submit" class="btn btn-primary"><i class="fas fa-calendar-check"></i> Book Appointment</button>
                <a href="/appointments" class="btn btn-secondary">Cancel</a>
            </div>
        </form>
    </div>
</div>

<script>
// Set minimum date to today using JavaScript
document.addEventListener('DOMContentLoaded', function() {
    var today = new Date().toISOString().split('T')[0];
    document.getElementById('appt_date').setAttribute('min', today);
    document.getElementById('appt_date').value = today;
});
</script>
{% endblock %}
```

# STEP 4: CREATE VIEW APPOINTMENT PAGE

sudo vim app/templates/appointments_view.html

```bash 
{% extends "base_dashboard.html" %}
{% block title %}Appointment Details{% endblock %}
{% block content %}
<div class="d-flex justify-content-between align-items-center mb-4">
    <h2><i class="fas fa-calendar-alt"></i> Appointment #{{ appointment.id }}</h2>
    <a href="/appointments" class="btn btn-secondary"><i class="fas fa-arrow-left"></i> Back</a>
</div>

<div class="row">
    <div class="col-md-6">
        <div class="card mb-4">
            <div class="card-body">
                <h5><i class="fas fa-user"></i> Patient Information</h5>
                <hr>
                <p><strong>Name:</strong> {{ appointment.patient_name }}</p>
                <p><strong>Patient ID:</strong> {{ patient.patient_id if patient else 'N/A' }}</p>
                <p><strong>Phone:</strong> {{ appointment.patient_phone or 'N/A' }}</p>
                <p><strong>Email:</strong> {{ appointment.patient_email or 'N/A' }}</p>
                {% if patient %}
                <a href="/patients/view/{{ patient.id }}" class="btn btn-sm btn-info"><i class="fas fa-eye"></i> View Full Profile</a>
                {% endif %}
            </div>
        </div>
    </div>

    <div class="col-md-6">
        <div class="card mb-4">
            <div class="card-body">
                <h5><i class="fas fa-calendar-check"></i> Appointment Details</h5>
                <hr>
                <p><strong>Date:</strong> {{ appointment.appointment_date.strftime('%Y-%m-%d') }}</p>
                <p><strong>Time:</strong> {{ appointment.appointment_time }}</p>
                <p><strong>Doctor:</strong> {{ appointment.doctor }}</p>
                <p><strong>Department:</strong> {{ appointment.department or 'N/A' }}</p>
                <p><strong>Status:</strong> <span class="badge badge-{{ appointment.status }}">{{ appointment.status|capitalize }}</span></p>
            </div>
        </div>
    </div>
</div>

<div class="card mb-4">
    <div class="card-body">
        <h5><i class="fas fa-notes-medical"></i> Reason for Visit</h5>
        <hr>
        <p>{{ appointment.reason }}</p>
        
        {% if appointment.notes %}
        <h5 class="mt-4"><i class="fas fa-sticky-note"></i> Additional Notes</h5>
        <hr>
        <p>{{ appointment.notes }}</p>
        {% endif %}
    </div>
</div>

<!-- UPDATE STATUS -->
<div class="card">
    <div class="card-body">
        <h5><i class="fas fa-tasks"></i> Update Status</h5>
        <hr>
        <form method="POST" action="/appointments/update-status/{{ appointment.id }}" class="d-flex gap-2">
            <select name="status" class="form-control" style="max-width: 200px;">
                <option value="pending" {% if appointment.status == 'pending' %}selected{% endif %}>Pending</option>
                <option value="confirmed" {% if appointment.status == 'confirmed' %}selected{% endif %}>Confirmed</option>
                <option value="completed" {% if appointment.status == 'completed' %}selected{% endif %}>Completed</option>
                <option value="cancelled" {% if appointment.status == 'cancelled' %}selected{% endif %}>Cancelled</option>
            </select>
            <button type="submit" class="btn btn-primary"><i class="fas fa-save"></i> Update Status</button>
        </form>
    </div>
</div>
{% endblock %}
```

# STEP 5: UPDATE BASE_DASHBOARD.HTML (FIX SIDEBAR)

sudo vim app/templates/base_dashboard.html

```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}HealthClinic{% endblock %}</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
    <style>
        body { background-color: #f8f9fa; }
        .sidebar { position: fixed; top: 0; left: 0; height: 100vh; width: 250px; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; overflow-y: auto; z-index: 1000; }
        .sidebar-header { padding: 1.5rem; text-align: center; border-bottom: 1px solid rgba(255,255,255,0.2); }
        .sidebar-menu { list-style: none; padding: 0; margin: 1rem 0; }
        .sidebar-menu a { display: block; padding: 1rem 1.5rem; color: rgba(255,255,255,0.8); text-decoration: none; transition: all 0.3s; }
        .sidebar-menu a:hover, .sidebar-menu a.active { background: rgba(255,255,255,0.1); color: white; border-left: 3px solid white; }
        .sidebar-menu i { width: 20px; margin-right: 10px; }
        .main-content { margin-left: 250px; padding: 2rem; min-height: 100vh; }
        .top-bar { background: white; padding: 1rem 2rem; border-radius: 10px; margin-bottom: 2rem; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .card { border: none; border-radius: 10px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .badge-pending { background: #ffc107; color: white; }
        .badge-confirmed { background: #28a745; color: white; }
        .badge-cancelled { background: #dc3545; color: white; }
        .badge-completed { background: #6c757d; color: white; }
    </style>
</head>
<body>
<div class="sidebar">
    <div class="sidebar-header">
        <i class="fas fa-hospital-alt fa-2x mb-2"></i>
        <h4>HealthClinic</h4>
    </div>
    <ul class="sidebar-menu">
        <li><a href="/dashboard"><i class="fas fa-tachometer-alt"></i> Dashboard</a></li>
        <li><a href="/patients"><i class="fas fa-users"></i> Patients</a></li>
        <li><a href="/appointments"><i class="fas fa-calendar-alt"></i> Appointments</a></li>
        <li><a href="/logout"><i class="fas fa-sign-out-alt"></i> Logout</a></li>
    </ul>
</div>
<div class="main-content">
    <div class="top-bar">
        <strong>{{ current_user.full_name }}</strong> | <small class="text-muted">{{ current_user.role|capitalize }}</small>
    </div>
    {% with messages = get_flashed_messages(with_categories=true) %}
        {% if messages %}
            {% for category, message in messages %}
                <div class="alert alert-{{ 'danger' if category == 'error' else category }} alert-dismissible fade show">
                    {{ message }}
                    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                </div>
            {% endfor %}
        {% endif %}
    {% endwith %}
    {% block content %}{% endblock %}
</div>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

# Adding appointments_list

appointments_list.html

```bash
{% extends "base_dashboard.html" %}
{% block title %}Appointments | HealthClinic{% endblock %}
{% block content %}
<div class="d-flex justify-content-between align-items-center mb-4">
    <h2><i class="fas fa-calendar-alt"></i> Appointments</h2>
    <a href="/appointments/book" class="btn btn-primary"><i class="fas fa-calendar-plus"></i> Book Appointment</a>
</div>

<!-- FILTERS -->
<div class="card mb-4">
    <div class="card-body">
        <form method="GET" action="/appointments" class="row g-3">
            <div class="col-md-4">
                <label class="form-label">Status</label>
                <select name="status" class="form-control">
                    <option value="">All Status</option>
                    <option value="pending" {% if status_filter == 'pending' %}selected{% endif %}>Pending</option>
                    <option value="confirmed" {% if status_filter == 'confirmed' %}selected{% endif %}>Confirmed</option>
                    <option value="completed" {% if status_filter == 'completed' %}selected{% endif %}>Completed</option>
                    <option value="cancelled" {% if status_filter == 'cancelled' %}selected{% endif %}>Cancelled</option>
                </select>
            </div>
            <div class="col-md-4">
                <label class="form-label">Date</label>
                <input type="date" name="date" class="form-control" value="{{ date_filter }}">
            </div>
            <div class="col-md-4">
                <label class="form-label">&nbsp;</label>
                <button type="submit" class="btn btn-primary w-100"><i class="fas fa-filter"></i> Filter</button>
            </div>
        </form>
    </div>
</div>

<!-- APPOINTMENTS TABLE -->
{% if appointments %}
<div class="card">
    <div class="card-body">
        <div class="table-responsive">
            <table class="table table-hover">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Patient</th>
                        <th>Doctor</th>
                        <th>Date</th>
                        <th>Time</th>
                        <th>Department</th>
                        <th>Status</th>
                        <th>Actions</th>
                    </tr>
                </thead>
                <tbody>
                    {% for appt in appointments %}
                    <tr>
                        <td><strong>#{{ appt.id }}</strong></td>
                        <td><i class="fas fa-user"></i> {{ appt.patient_name }}</td>
                        <td><i class="fas fa-user-md"></i> {{ appt.doctor }}</td>
                        <td><i class="fas fa-calendar"></i> {{ appt.appointment_date.strftime('%Y-%m-%d') }}</td>
                        <td><i class="fas fa-clock"></i> {{ appt.appointment_time }}</td>
                        <td>{{ appt.department or 'N/A' }}</td>
                        <td>
                            <span class="badge badge-{{ appt.status }}">{{ appt.status|capitalize }}</span>
                        </td>
                        <td>
                            <a href="/appointments/view/{{ appt.id }}" class="btn btn-sm btn-info"><i class="fas fa-eye"></i></a>
                            <form method="POST" action="/appointments/delete/{{ appt.id }}" style="display:inline;" onsubmit="return confirm('Delete this appointment?');">
                                <button type="submit" class="btn btn-sm btn-danger"><i class="fas fa-trash"></i></button>
                            </form>
                        </td>
                    </tr>
                    {% endfor %}
                </tbody>
            </table>
        </div>
        <p class="text-muted mt-3">Total: {{ appointments|length }} appointment(s)</p>
    </div>
</div>
{% else %}
<div class="card">
    <div class="card-body text-center py-5">
        <i class="fas fa-calendar-times fa-3x text-muted mb-3"></i>
        <p class="text-muted">No appointments found.</p>
        <a href="/appointments/book" class="btn btn-primary">Book First Appointment</a>
    </div>
</div>
{% endif %}
{% endblock %}
```

# Add patients_list
/var/www/healthclinic/app/templates

```bash
{% extends "base_dashboard.html" %}
{% block title %}Patients | HealthClinic{% endblock %}

{% block content %}
<div class="d-flex justify-content-between align-items-center mb-4">
    <h2><i class="fas fa-users"></i> Patients</h2>
    <a href="/patients/add" class="btn btn-primary"><i class="fas fa-user-plus"></i> Add Patient</a>
</div>

<!-- SEARCH -->
<div class="card mb-4">
    <div class="card-body">
        <form method="GET" action="/patients" class="row g-3">
            <div class="col-md-10">
                <label class="form-label">Search Patient</label>
                <input type="text" name="search" class="form-control" placeholder="Search by name or patient ID"
                       value="{{ search }}">
            </div>
            <div class="col-md-2">
                <label class="form-label">&nbsp;</label>
                <button class="btn btn-primary w-100"><i class="fas fa-search"></i> Search</button>
            </div>
        </form>
    </div>
</div>

<!-- PATIENTS TABLE -->
{% if patients %}
<div class="card">
    <div class="card-body">
        <div class="table-responsive">
            <table class="table table-hover">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Patient ID</th>
                        <th>Name</th>
                        <th>Gender</th>
                        <th>Phone</th>
                        <th>Actions</th>
                    </tr>
                </thead>
                <tbody>
                    {% for p in patients %}
                    <tr>
                        <td>{{ p.id }}</td>
                        <td><strong>{{ p.patient_id }}</strong></td>
                        <td>{{ p.first_name }} {{ p.last_name }}</td>
                        <td>{{ p.gender }}</td>
                        <td>{{ p.phone }}</td>
                        <td>
                            <a href="/patients/view/{{ p.id }}" class="btn btn-sm btn-info">
                                <i class="fas fa-eye"></i>
                            </a>
                            <a href="/patients/edit/{{ p.id }}" class="btn btn-sm btn-warning">
                                <i class="fas fa-edit"></i>
                            </a>
                            <form method="POST" action="/patients/delete/{{ p.id }}" style="display:inline;" onsubmit="return confirm('Delete this patient?');">
                                <button type="submit" class="btn btn-sm btn-danger"><i class="fas fa-trash"></i></button>
                            </form>
                        </td>
                    </tr>
                    {% endfor %}
                </tbody>
            </table>
        </div>
        <p class="text-muted mt-3">Total: {{ patients|length }} patient(s)</p>
    </div>
</div>
{% else %}
<div class="card">
    <div class="card-body text-center py-5">
        <i class="fas fa-user-times fa-3x text-muted mb-3"></i>
        <p class="text-muted">No patients found.</p>
        <a href="/patients/add" class="btn btn-primary">Add First Patient</a>
    </div>
</div>
{% endif %}

{% endblock %}
```



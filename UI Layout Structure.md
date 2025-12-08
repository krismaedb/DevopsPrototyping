# ============================================================================
# WEB01 - COMPLETE SETUP: LANDING PAGE + AUTHENTICATION SYSTEM
# ============================================================================
# Description: Automated setup for HealthClinic web application
# Includes: Enhanced UI, Database Models, Authentication, User Management
# Password for all users: g3company!@#
# ============================================================================

cd /var/www/healthclinic

# Activate virtual environment
source venv/bin/activate

# Install Flask-Login
pip install flask-login

# ============================================================================
# PART 1: CREATE ENHANCED LANDING PAGE
# ============================================================================

cat > app/templates/index.html << 'EOF'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HealthClinic | Welcome</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
    <style>
        .card { transition: transform 0.3s ease, box-shadow 0.3s ease; }
        .card:hover { transform: translateY(-10px); box-shadow: 0 10px 20px rgba(0,0,0,0.15) !important; }
        .btn { transition: all 0.3s ease; }
        .btn:hover { transform: scale(1.05); }
        footer a { text-decoration: none; transition: opacity 0.3s ease; }
        footer a:hover { opacity: 0.8; }
    </style>
</head>
<body class="bg-light">
<nav class="navbar navbar-expand-lg navbar-dark bg-primary">
    <div class="container">
        <a class="navbar-brand fw-bold" href="/"><i class="fas fa-hospital-alt"></i> HealthClinic</a>
        <div class="d-flex">
            <a href="/appointment" class="btn btn-light btn-sm me-2"><i class="fas fa-calendar-plus"></i> Book Appointment</a>
            <a href="/login" class="btn btn-outline-light btn-sm"><i class="fas fa-user-lock"></i> Staff Login</a>
        </div>
    </div>
</nav>
<header class="py-5 bg-white shadow-sm">
    <div class="container text-center">
        <h1 class="fw-bold display-4">Your Health, Our Priority.</h1>
        <p class="lead text-muted">Modern, reliable healthcare powered by technology.</p>
        <a href="/appointment" class="btn btn-primary btn-lg mt-3"><i class="fas fa-calendar-check"></i> Book an Appointment</a>
    </div>
</header>
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
<section class="py-5 bg-light">
    <div class="container text-center">
        <h2 class="mb-5">Why Choose HealthClinic?</h2>
        <div class="row">
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
<section class="py-5 bg-white">
    <div class="container">
        <div class="row">
            <div class="col-md-6 mb-4">
                <h4><i class="fas fa-map-marker-alt text-primary"></i> Location</h4>
                <p class="ms-4">123 Main Street<br>Winnipeg, MB R3C 1A5<br>Canada</p>
            </div>
            <div class="col-md-6 mb-4">
                <h4><i class="fas fa-clock text-primary"></i> Clinic Hours</h4>
                <p class="ms-4">Monday - Friday: 8:00 AM - 6:00 PM<br>Saturday: 9:00 AM - 2:00 PM<br>Sunday: Closed</p>
            </div>
        </div>
    </div>
</section>
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
                <p><i class="fas fa-phone"></i> (204) 555-0123<br><i class="fas fa-envelope"></i> info@healthclinic.local</p>
            </div>
        </div>
        <hr class="bg-white">
        <p class="text-center mb-0">© 2025 HealthClinic • MITT Capstone Project • All Rights Reserved</p>
    </div>
</footer>
</body>
</html>
EOF

# ============================================================================
# PART 2: CREATE DATABASE MODELS
# ============================================================================

cat > app/models.py << 'EOF'
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
    role = db.Column(db.String(20), nullable=False)
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
    status = db.Column(db.String(20), default='pending')
    notes = db.Column(db.Text)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    def __repr__(self):
        return f'<Appointment {self.patient_name} - {self.appointment_date}>'
EOF

# ============================================================================
# PART 3: CONFIGURE FLASK AUTHENTICATION
# ============================================================================

cat > app/__init__.py << 'EOF'
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager

db = SQLAlchemy()
login_manager = LoginManager()

def create_app():
    app = Flask(__name__)
    app.config['SECRET_KEY'] = 'healthclinic2025-secret-key-change-in-production'
    app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://webadmin:T%40ylorSwift13@127.0.0.1:5432/healthclinic'
    app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
    db.init_app(app)
    login_manager.init_app(app)
    login_manager.login_view = 'main.login'
    login_manager.login_message = 'Please log in to access this page.'
    @login_manager.user_loader
    def load_user(user_id):
        from .models import User
        return User.query.get(int(user_id))
    from .routes import main_bp
    app.register_blueprint(main_bp)
    return app
EOF

# ============================================================================
# PART 4: CREATE LOGIN PAGE
# ============================================================================

cat > app/templates/login.html << 'EOF'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Staff Login | HealthClinic</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
    <style>
        body { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); min-height: 100vh; display: flex; align-items: center; justify-content: center; }
        .login-card { background: white; border-radius: 15px; box-shadow: 0 10px 40px rgba(0,0,0,0.2); overflow: hidden; max-width: 900px; width: 100%; }
        .login-left { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; padding: 3rem; display: flex; flex-direction: column; justify-content: center; }
        .login-right { padding: 3rem; }
        .form-control:focus { border-color: #667eea; box-shadow: 0 0 0 0.2rem rgba(102, 126, 234, 0.25); }
        .btn-login { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); border: none; padding: 0.75rem; font-weight: bold; }
        .btn-login:hover { transform: translateY(-2px); box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4); }
        .role-badge { display: inline-block; padding: 0.5rem 1rem; margin: 0.25rem; border-radius: 20px; background: rgba(255,255,255,0.2); font-size: 0.9rem; }
    </style>
</head>
<body>
<div class="container">
    <div class="login-card row g-0">
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
        <div class="col-md-7 login-right">
            <div class="mb-4">
                <h3 class="fw-bold mb-2">Welcome Back</h3>
                <p class="text-muted">Sign in to access your dashboard</p>
            </div>
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
            <form method="POST" action="{{ url_for('main.login') }}">
                <div class="mb-3">
                    <label class="form-label"><i class="fas fa-user"></i> Username</label>
                    <input type="text" name="username" class="form-control form-control-lg" placeholder="Enter your username" required autofocus>
                </div>
                <div class="mb-3">
                    <label class="form-label"><i class="fas fa-lock"></i> Password</label>
                    <input type="password" name="password" class="form-control form-control-lg" placeholder="Enter your password" required>
                </div>
                <div class="mb-3 form-check">
                    <input type="checkbox" name="remember" class="form-check-input" id="remember">
                    <label class="form-check-label" for="remember">Remember me</label>
                </div>
                <button type="submit" class="btn btn-primary btn-login w-100 btn-lg"><i class="fas fa-sign-in-alt"></i> Sign In</button>
            </form>
            <hr class="my-4">
            <div class="text-center">
                <a href="/" class="text-decoration-none"><i class="fas fa-arrow-left"></i> Back to Homepage</a>
            </div>
            <div class="text-center mt-3">
                <small class="text-muted"><i class="fas fa-info-circle"></i> Having trouble logging in? Contact IT Support</small>
            </div>
        </div>
    </div>
</div>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
EOF

# ============================================================================
# PART 5: UPDATE ROUTES WITH AUTHENTICATION
# ============================================================================

cat > app/routes.py << 'EOF'
from flask import Blueprint, render_template, request, redirect, url_for, flash
from flask_login import login_user, logout_user, login_required, current_user
from datetime import datetime
from . import db
from .models import User, Patient, Appointment

main_bp = Blueprint('main', __name__)

@main_bp.route('/')
def index():
    return render_template('index.html')

@main_bp.route('/appointment')
def appointment():
    return render_template('appointment.html')

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

@main_bp.route('/dashboard')
@login_required
def dashboard():
    total_patients = Patient.query.count()
    total_appointments = Appointment.query.count()
    pending_appointments = Appointment.query.filter_by(status='pending').count()
    today_appointments = Appointment.query.filter_by(appointment_date=datetime.utcnow().date()).count()
    recent_appointments = Appointment.query.order_by(Appointment.created_at.desc()).limit(5).all()
    return render_template('dashboard.html',
                         total_patients=total_patients,
                         total_appointments=total_appointments,
                         pending_appointments=pending_appointments,
                         today_appointments=today_appointments,
                         recent_appointments=recent_appointments)
EOF

# ============================================================================
# PART 6: CREATE USER MANAGEMENT SCRIPT
# ============================================================================

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
        print("\n" + "="*80)
        print("CURRENT USERS")
        print("="*80)
        print(f"{'Username':<20} {'Role':<12} {'Full Name':<30} {'Status'}")
        print("-"*80)
        for user in users:
            status = "Active" if user.is_active else "Inactive"
            print(f"{user.username:<20} {user.role:<12} {user.full_name:<30} {status}")
        print("-"*80)
        print(f"Total Users: {len(users)}\n")

def reset_all_passwords():
    app = create_app()
    with app.app_context():
        users = User.query.all()
        print("\n🔑 Resetting passwords to: g3company!@#\n")
        for user in users:
            user.set_password(DEFAULT_PASSWORD)
            print(f"✅ Reset: {user.username}")
        db.session.commit()
        print(f"\n✅ All {len(users)} passwords reset!\n")

def main():
    if len(sys.argv) > 1:
        if sys.argv[1] == '--list':
            list_users()
            return
        elif sys.argv[1] == '--reset-passwords':
            reset_all_passwords()
            return
    print("\n" + "="*80)
    print("HEALTHCLINIC USER CREATOR")
    print("="*80)
    print(f"Default Password: {DEFAULT_PASSWORD}\n")
    default_users = [
        {'username': 'admin', 'email': 'admin@healthclinic.local', 'full_name': 'System Administrator', 'role': 'admin', 'phone': '204-555-0100'},
        {'username': 'psantos', 'email': 'psantos@healthclinic.local', 'full_name': 'Paul Santos', 'role': 'it', 'phone': '204-555-0101'},
        {'username': 'dr.johnson', 'email': 'sjohnson@healthclinic.local', 'full_name': 'Dr. Sarah Johnson', 'role': 'doctor', 'phone': '204-555-0102'},
        {'username': 'nurse.maria', 'email': 'maria@healthclinic.local', 'full_name': 'Maria Gonzales', 'role': 'nurse', 'phone': '204-555-0103'},
        {'username': 'admin.billing', 'email': 'billing@healthclinic.local', 'full_name': 'Billing Administrator', 'role': 'admin', 'phone': '204-555-0104'},
        {'username': 'dr.chen', 'email': 'mchen@healthclinic.local', 'full_name': 'Dr. Michael Chen', 'role': 'doctor', 'phone': '204-555-0105'},
        {'username': 'nurse.emily', 'email': 'emily@healthclinic.local', 'full_name': 'Emily Rodriguez', 'role': 'nurse', 'phone': '204-555-0106'},
    ]
    created = 0
    skipped = 0
    for user_data in default_users:
        if create_user(**user_data):
            created += 1
        else:
            skipped += 1
    print("\n" + "-"*80)
    print(f"✅ Created: {created} | ⚠️  Skipped: {skipped}")
    print("-"*80 + "\n")
    list_users()

if __name__ == '__main__':
    main()
EOF

chmod +x app/create_user.py
sudo chown webadmin:www-data app/create_user.py

# ============================================================================
# PART 7: CREATE DATABASE TABLES
# ============================================================================

python3 << 'PYEOF'
from app import create_app, db
app = create_app()
with app.app_context():
    db.create_all()
    print("✅ Database tables created successfully!")
PYEOF

# ============================================================================
# PART 8: CREATE DEFAULT USERS
# ============================================================================

python3 app/create_user.py

# ============================================================================
# PART 9: RESTART APACHE
# ============================================================================

sudo systemctl restart apache2

# ============================================================================
# SETUP COMPLETE
# ============================================================================

echo ""
echo "============================================================================"
echo "✅ WEB01 SETUP COMPLETE!"
echo "============================================================================"
echo ""
echo "Landing Page:  http://10.10.40.30"
echo "Staff Login:   http://10.10.40.30/login"
echo ""
echo "Default Users (Password: g3company!@#):"
echo "  - admin          (System Administrator)"
echo "  - psantos        (Paul Santos - IT)"
echo "  - dr.johnson     (Dr. Sarah Johnson - Doctor)"
echo "  - nurse.maria    (Maria Gonzales - Nurse)"
echo "  - admin.billing  (Billing Administrator)"
echo "  - dr.chen        (Dr. Michael Chen - Doctor)"
echo "  - nurse.emily    (Emily Rodriguez - Nurse)"
echo ""
echo "User Management Commands:"
echo "  python3 app/create_user.py              # Create users"
echo "  python3 app/create_user.py --list       # List all users"
echo "  python3 app/create_user.py --reset-passwords  # Reset passwords"
echo ""
echo "============================================================================"

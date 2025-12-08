## ✅ 1. UI Layout Structure

Make sure your folder layout looks like this:

/var/www/healthclinic  
│── app/  
│   ├── __init__.py  
│   ├── models.py  
│   ├── routes.py  
│   ├── static/  
│   └── templates/  
│       └── index.html  
│  
├── venv/  
└── healthclinic.wsgi

If "templates" or "static" does not exist, create them:

mkdir /var/www/healthclinic/app/templates  
mkdir /var/www/healthclinic/app/static

---

## ✅ 2. Update routes.py for Landing Page

Open `/var/www/healthclinic/app/routes.py`:

```bash
from flask import Blueprint, render_template

main = Blueprint('main', __name__)

# --- Landing Page ---
@main.route('/')
def home():
    return render_template('index.html')

# --- Appointment Booking Page ---
@main.route('/book')
def book_appointment():
    return render_template('book.html')

# --- Login Page (placeholder) ---
@main.route('/login')
def login():
    return render_template('login.html')

# --- Dashboard Page (placeholder for staff only) ---
@main.route('/dashboard')
def dashboard():
    return render_template('dashboard.html')
```

---

## ✅ 3. Register Blueprint in __init__.py

Open `/var/www/healthclinic/app/__init__.py`:

```

from flask import Flask
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

def create_app():
    app = Flask(__name__)
    app.config["SQLALCHEMY_DATABASE_URI"] = (
        "postgresql://webadmin:T%40ylorSwift13@127.0.0.1:5432/healthclinic"
    )
    app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False

    db.init_app(app)

    from .routes import main
    app.register_blueprint(main)

    return app

```

---

## ✅ 4. Create Landing Page (index.html)

Inside `/var/www/healthclinic/app/templates/index.html`:

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HealthClinic</title>

    <!-- Free Bootstrap 5 CDN -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
</head>

<body class="bg-light">
    <nav class="navbar navbar-dark bg-primary">
        <div class="container">
            <a class="navbar-brand" href="/">🏥 HealthClinic</a>
        </div>
    </nav>

    <header class="text-center py-5 bg-white shadow-sm">
        <h1 class="mb-3">Welcome to HealthClinic</h1>
        <p class="lead">Your trusted neighborhood medical care provider</p>
        <a href="#" class="btn btn-primary btn-lg">Book Appointment</a>
    </header>

    <section class="container py-5">
        <h2 class="text-center mb-4">Our Services</h2>

        <div class="row text-center">
            <div class="col-md-4 mb-4">
                <div class="p-4 bg-white shadow-sm rounded">
                    <h4>General Checkup</h4>
                    <p>Routine medical exams for all ages.</p>
                </div>
            </div>

            <div class="col-md-4 mb-4">
                <div class="p-4 bg-white shadow-sm rounded">
                    <h4>Pediatrics</h4>
                    <p>Professional care dedicated for children.</p>
                </div>
            </div>

            <div class="col-md-4 mb-4">
                <div class="p-4 bg-white shadow-sm rounded">
                    <h4>Laboratory Tests</h4>
                    <p>Fast and reliable diagnostic testing.</p>
                </div>
            </div>
        </div>
    </section>

    <footer class="text-center py-3 bg-primary text-white">
        © 2025 HealthClinic. All rights reserved.
    </footer>
</body>
</html>

---

## ✅ 5. Reload Apache

sudo systemctl restart apache2

---

🎉 **END RESULT**

Visiting:

http://your-server-ip/

Now displays:  
✔ Bootstrap-based professional landing page  
✔ Responsive UI  
✔ Buttons, services, navbar, footer  
✔ Phase 1 completed

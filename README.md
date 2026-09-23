<div align="center">

# 🎓 Schola

**A web-based scholarship management system that brings transparency and efficiency to educational administration.**

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Django](https://img.shields.io/badge/Django-5.x-092E20?logo=django&logoColor=white)](https://www.djangoproject.com/)
[![Database](https://img.shields.io/badge/Database-SQLite3-003B57?logo=sqlite&logoColor=white)](https://www.sqlite.org/)
[![Pillow](https://img.shields.io/badge/Pillow-%3E%3D10.0-yellow)](https://python-pillow.org/)
[![Status](https://img.shields.io/badge/Status-In%20Development-orange)](#)

[🎥 Video Demo](https://drive.google.com/file/d/1c0fToAYp1kD6zXMx_S88OORNQUDkDkBM/view?usp=sharing) · [🎨 Figma Design System](https://www.figma.com/design/QEcHPgF8q3xFsjlfIymJYk/Sofeng?node-id=0-1&t=EmsOcz8CpI96rPnQ-1)

</div>

---

## Table of Contents

1. [Overview](#overview)
2. [Links & Resources](#links--resources)
3. [Key Features](#key-features)
4. [System Architecture & Tech Stack](#system-architecture--tech-stack)
5. [Directory Structure](#directory-structure)
6. [Local Installation & Setup](#local-installation--setup)
7. [Usage & Workflow Guide](#usage--workflow-guide)
8. [Known Limitations](#known-limitations)
9. [License](#license)

---

## Overview

**Schola** (*"Sistem Informasi Pendaftaran Beasiswa Berbasis Web"*) is a full-stack Django web application that digitizes the entire scholarship lifecycle — from publishing opportunities to reviewing applicant documents — for institutions that currently manage this process manually or across disconnected spreadsheets and forms.

It addresses two connected problems:

- **For students**, finding and applying to scholarships is often scattered across flyers, PDFs, and emails, with no single place to track application status or required documents.
- **For scholarship administrators**, reviewing applicants, verifying uploaded documents, and updating statuses by hand is slow, error-prone, and hard to audit.

Schola solves this with a single platform offering **two distinct experiences built on one shared data model**:

| Audience | Primary Use Cases |
|---|---|
| 🎓 **Students** | Browse & search scholarships, apply, upload required documents (CV, transcript, motivation letter), track application status, receive notifications, manage their profile |
| 🛠️ **Administrators** | Publish and manage scholarship listings they own, review applicants for each listing, inspect submitted documents, and accept/reject applications |

## Links & Resources

| Resource | Link |
|---|---|
| 🎥 **Video Demo** | [Watch on Google Drive](https://drive.google.com/file/d/1c0fToAYp1kD6zXMx_S88OORNQUDkDkBM/view?usp=sharing) |
| 🎨 **Figma Design System / Mockups** | [View on Figma](https://www.figma.com/design/QEcHPgF8q3xFsjlfIymJYk/Sofeng?node-id=0-1&t=EmsOcz8CpI96rPnQ-1) |

## Key Features

### 👤 Authentication & Accounts
- Custom email-based user model (`CustomUser`) with `student` / `admin` roles, replacing Django's default username login
- Registration, login, logout, and profile editing flows
- Role-aware redirects after login (students land on the scholarship list, admins land on their management dashboard)
- Role-gated access control via reusable `@student_required` and `@admin_required` view decorators, which also block inactive accounts

### 🎓 Scholarship Management (Student side)
- Browse all published scholarships with **search** (by name/provider) and **filters** (by country and degree level: S1/Undergraduate, S2/Master, S3/Doctoral)
- View detailed scholarship pages (requirements, benefits, deadline, provider)
- Prevents duplicate applications — a student can only apply once per scholarship (enforced at the database level)
- Automatic deadline validation — scholarships cannot be created with a deadline in the past, and an `is_open` property flags expired listings

### 🛠️ Scholarship Management (Admin side)
- Full CRUD (create, edit, delete with confirmation) for scholarships owned by the logged-in admin
- Admin dashboard listing only the scholarships that admin created (`created_by` scoping)

### 📄 Application & Document Workflow
- One-click apply flow that creates an `Application` record and an automatic confirmation notification
- Document upload with **server-side validation**:
  - Accepted document types: CV, Transcript, Motivation Letter
  - File type restricted to PDF, JPG, JPEG, PNG (validated by both MIME type and extension)
  - 5 MB maximum file size
  - One document per type per application (prevents duplicate uploads)
- File size is captured and stored automatically on save
- Student-facing **status dashboard** summarizing Pending / Accepted / Rejected counts
- Admin **applicant list & detail views**, scoped to applications on scholarships that admin owns
- Admin can update an application's status (Pending → Accepted/Rejected), reflected instantly for the student

### 🔔 Notifications
- Per-user notification model with types (`info`, `success`, `warning`, `document`), read/unread state, and an optional action URL
- Notifications are automatically generated on key events (e.g., successful application submission)
- Dedicated notifications page for students

### 📚 Resource Hub
- A dedicated `resources` app scaffolding a student resource center — guides, templates, FAQs, success stories, and live sessions — with routing already in place for a category library and article detail pages

### 🎨 Fully Custom Front End
- Hand-built HTML/CSS templates (no CSS framework dependency) organized per feature area, including a dedicated admin panel theme and a student-facing theme
- A large `static/` prototype library (`admin_html`, `components`, `css`) mirroring the Figma design system, used as the source for the Django templates

## System Architecture & Tech Stack

### Architecture

Schola is a classic **server-rendered Django monolith** using Django's MVT (Model–View–Template) pattern — there is no separate frontend framework or REST API layer; views render server-side HTML templates directly.

```text
Browser
   │
   ▼
schola/urls.py  ──►  apps/{users,scholarships,applications,resources}/urls.py
                              │
                              ▼
                     views.py  (role-gated via apps/utils/decorators.py)
                              │
                 ┌────────────┼────────────┐
                 ▼            ▼             ▼
            forms.py      models.py     templates/*.html
                              │
                              ▼
                      SQLite3 (db.sqlite3)
```

### Core Technologies

| Layer | Technology |
|---|---|
| Backend framework | [Django](https://www.djangoproject.com/) `>=5.0,<6.0` |
| Language | Python 3.10+ |
| Database | SQLite3 (default Django `django.db.backends.sqlite3`) |
| Image/File handling | [Pillow](https://python-pillow.org/) `>=10.0` (required for Django `ImageField`/`FileField` support) |
| Authentication | Custom user model (`AUTH_USER_MODEL = 'users.CustomUser'`) with a custom `UserManager` |
| Templating | Django Template Language (DTL), template inheritance via `templates/partials/` |
| Frontend | Hand-authored HTML/CSS (no JS framework); vanilla `static/js/navbar.js` for interactivity |
| Design source | Figma (see [Links & Resources](#links--resources)) |
| Admin interface | Django Admin, customized per app (`admin.py` in each app) |

### Data Model Summary

| App | Model | Purpose |
|---|---|---|
| `users` | `CustomUser` | Email-based auth, `role` (`student`/`admin`), phone (regex-validated), timestamps |
| `scholarships` | `Scholarship` | Name, provider, deadline, country, level, requirements, benefits, owning admin (`created_by`) |
| `applications` | `Application` | Links a student to a scholarship, tracks `status` (pending/accepted/rejected); unique per (user, scholarship) |
| `applications` | `Document` | Uploaded file per application, typed (CV/Transcript/Motivation Letter), unique per (application, type), auto-captures file size |
| `applications` | `Notification` | Per-user notification with type, title, message, read state, optional action URL |

> [!NOTE]
> The `resources` app currently has no models — its views render templates with placeholder/empty context (`featured_articles`, `templates`, `faqs`, etc.), meaning the resource hub is scaffolded but not yet backed by persisted data.

## Directory Structure

```text
Schola-System/
├── apps/
│   ├── users/            # Custom auth model, registration/login/profile
│   ├── scholarships/     # Scholarship CRUD (student browse + admin manage)
│   ├── applications/     # Applications, document uploads, notifications
│   ├── resources/        # Student resource hub (views scaffolded, no models yet)
│   └── utils/
│       └── decorators.py # @student_required / @admin_required access control
├── schola/
│   ├── settings.py       # Project configuration
│   ├── urls.py            # Root URL routing
│   └── wsgi.py / asgi.py
├── templates/             # Django templates actually rendered by the app
│   ├── admin_panel/
│   ├── applications/
│   ├── partials/          # Shared navbar/sidebar/footer includes
│   ├── resources/
│   ├── scholarships/
│   └── users/
├── static/                 # CSS, JS, images, and Figma-derived HTML prototypes
│   ├── admin_css/ , admin_html/     # Admin panel prototype source
│   ├── components/                  # Student-facing HTML component prototypes
│   └── css/ , js/ , assets/         # Stylesheets, scripts, images used by templates
├── manage.py               # Django management entry point
├── seed.py                 # Creates demo admin/student accounts + a sample scholarship
├── requirements.txt
└── README.md
```

## Local Installation & Setup

### Prerequisites

- **Python 3.10+** (required by Django 5.x)
- `pip`
- Git

### 1. Clone the repository

```bash
git clone <your-repository-url>
cd Schola-System
```

### 2. Create and activate a virtual environment

```bash
# macOS / Linux
python3 -m venv venv
source venv/bin/activate

# Windows (PowerShell)
python -m venv venv
venv\Scripts\Activate.ps1
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

This installs `Django>=5.0,<6.0` and `Pillow>=10.0`.

### 4. Environment configuration

> [!IMPORTANT]
> The current codebase does **not** use a `.env` file or `python-dotenv` — `SECRET_KEY`, `DEBUG`, and `ALLOWED_HOSTS` are hardcoded directly in `schola/settings.py` for local/academic use. No environment variables are required to run the project as-is.
>
> Before deploying this project anywhere public, it is strongly recommended to externalize these values (e.g. with `python-dotenv` or `django-environ`):
>
> | Setting | Current value in `settings.py` | Recommended change |
> |---|---|---|
> | `SECRET_KEY` | Hardcoded placeholder key | Load from an environment variable / secrets manager |
> | `DEBUG` | `True` | `False` in production |
> | `ALLOWED_HOSTS` | `['*']` | Restrict to your actual domain(s) |

### 5. Apply database migrations

Schola uses SQLite3 by default, so no external database server is required.

```bash
python manage.py migrate
```

### 6. (Optional) Create an admin account

Either create a superuser interactively:

```bash
python manage.py createsuperuser
```

or seed demo data (one admin, one student, and one sample scholarship) with the included script:

```bash
python manage.py shell < seed.py
# or, from the project root:
python seed.py
```

This creates:

| Role | Email | Password |
|---|---|---|
| Admin | `admin@schola.com` | `admin123` |
| Student | `student@schola.com` | `student123` |

> [!WARNING]
> These are demo credentials for local development only — never seed these into a production database.

### 7. Run the development server

```bash
python manage.py runserver
```

The app will be available at:

```
http://127.0.0.1:8000/
```

Visiting `/` redirects to the login page (`/auth/login/`). The Django admin is available at `/admin/`.

## Usage & Workflow Guide

### As a Student

1. **Register** at `/auth/register/` or **log in** at `/auth/login/`.
2. Land on the **scholarship list** (`/scholarships/`) — search by keyword or filter by country/degree level.
3. Open a scholarship's **detail page** to review requirements, benefits, and deadline.
4. Click **Apply** to submit an application (one application per scholarship is enforced automatically).
5. Go to the application's **upload page** to submit required documents (CV, transcript, motivation letter) — each file is validated for type and size before it's accepted.
6. Track progress on the **Status** page (`/applications/status/`), which summarizes Pending / Accepted / Rejected applications.
7. Check the **Notifications** page for updates (e.g., submission confirmations).
8. Manage personal details anytime from the **Profile** page (`/auth/profile/`).

### As an Administrator

1. Log in with an admin account — you're redirected to the **admin scholarship dashboard** (`/scholarships/manage/`), scoped to scholarships you created.
2. **Create** a new scholarship listing via `/scholarships/manage/create/`, or **edit**/**delete** existing ones.
3. Go to the **applicant list** (`/applications/admin/`) to see everyone who applied to your scholarships, with live Pending/Accepted/Rejected counts.
4. Open an **applicant's detail page** to review their submitted documents.
5. **Update the application status** (accept or reject) — the student sees the updated status immediately on their end.
6. Use the standard **Django Admin** (`/admin/`) for lower-level data management (users, scholarships, applications, documents, notifications) if needed.

## Known Limitations

Based on the current state of the codebase, these areas are present in the UI but not yet fully wired up on the backend:

- **Saved scholarships** — the student "Saved" page renders (`scholarships:saved`), but there is no model or logic yet to persist which scholarships a student has bookmarked.
- **Resources hub** — routes and templates exist for articles, categories, and template downloads, but the `resources` app has no models yet; all list views currently render with empty placeholder data.
- **"Mark all as read"** for notifications is wired to a view, but the underlying query should be verified before relying on it.

## License

No license file is currently included in this repository. All rights are reserved to the authors unless a license is added.

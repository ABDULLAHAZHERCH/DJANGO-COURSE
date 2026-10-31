# Practical Lab Guide 02: Creating a Django Project & App Structure

## 🎯 Learning Objectives

By the end of this lab, you will be able to:

1. Differentiate between a **Django Project** and a **Django Application**.
2. Use `django-admin startproject` to generate the project core.
3. Understand the role of core configuration files (`settings.py`, `urls.py`, `wsgi.py`, `asgi.py`, `manage.py`).
4. Create a modular Django app using `python manage.py startapp`.
5. Run database migrations and start the built-in development server.

## 📑 Module Overview

| Section | Topic | Practical Focus |
| :--- | :--- | :--- |
| **01** | Projects vs. Applications | Architectural distinction and best practices |
| **02** | Creating the Django Project | Executing `django-admin startproject` |
| **03** | Understanding Core Files | File-by-file configuration analysis |
| **04** | Creating & Registering an App | `python manage.py startapp` & `INSTALLED_APPS` |
| **05** | Migrations & Development Server | Database setup and web server execution |
| **06** | Student Hands-On Tasks | Practice exercises & common troubleshooting |

---

## 01. Conceptual Foundation: Projects vs. Applications

In Django, there is a clear distinction between a **Project** and an **Application (App)**:

```
+-----------------------------------------------------------------------+
|                       DJANGO PROJECT CONTAINER                        |
|                     (e.g., student_management)                        |
|                                                                       |
|  +------------------------+  +------------------------------------+  |
|  |     Project Core       |  |             Students App           |  |
|  |  (settings, urls, etc) |  |   (models, views, templates, etc)  |  |
|  +------------------------+  +------------------------------------+  |
|                                                                       |
|  +------------------------+  +------------------------------------+  |
|  |     Courses App        |  |          Authentication App        |  |
|  |  (models, views, etc)  |  |   (models, views, templates, etc)  |  |
|  +------------------------+  +------------------------------------+  |
+-----------------------------------------------------------------------+
```

* **Project:** The complete web application configuration container. It houses global settings, main routing, database credentials, and registered apps.
* **App:** A self-contained Python package that handles a specific set of features (e.g., a *students* app, a *blog* app, an *authentication* app). An app can be reused across different Django projects.

---

## 02. Creating the Django Project

### 2.1 Preparing the Environment

Ensure your virtual environment is active before running any commands:

```bash
cd ~/projects/student_management
source venv/bin/activate
```

Verify `(venv)` is displayed in your terminal prompt.

### 2.2 Initializing the Project Structure

Run `django-admin startproject` with a trailing dot (`.`):

```bash
django-admin startproject config .
```

> ⚠️ **Crucial Syntax Note:** The trailing dot (`.`) tells Django to create the project files in the **current directory** rather than nesting them inside an extra folder layer.

### 2.3 Inspecting the Project Tree

Run `tree` to inspect the generated layout:

```bash
tree -I "venv|__pycache__"
```

*Expected Structure:*

```
.
├── config/
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── manage.py
└── requirements.txt
```

---

## 03. Understanding Core Project Files

| File | Purpose & Functions |
| :--- | :--- |
| `manage.py` | Command-line utility for project interactions (running server, migrations, shell, creating superusers). |
| `config/settings.py` | Master configuration file containing database credentials, security keys, installed apps, and static asset settings. |
| `config/urls.py` | Top-level URL routing configuration. Acts as the table of contents for routing requests to app views. |
| `config/wsgi.py` | Entry point for **WSGI** web servers (e.g., Gunicorn, uWSGI) used in standard synchronous production deployments. |
| `config/asgi.py` | Entry point for **ASGI** web servers (e.g., Daphne, Uvicorn) used in asynchronous web applications (WebSockets, async views). |

---

## 04. Creating and Registering a Django App

Now we will create the feature-specific app for managing students.

### 4.1 Generating the App Structure

Run `manage.py startapp` to create an app named `students`:

```bash
python manage.py startapp students
```

Inspect the updated directory tree:

```bash
tree -I "venv|__pycache__"
```

*Updated Structure:*

```
.
├── config/
│   ├── ...
├── students/
│   ├── admin.py          # Admin interface registration
│   ├── apps.py           # App configuration class
│   ├── __init__.py
│   ├── migrations/       # Database migration tracking files
│   │   └── __init__.py
│   ├── models.py         # Database schema classes
│   ├── tests.py          # Unit tests for the app
│   └── views.py          # HTTP request handlers/logic
├── manage.py
└── requirements.txt
```

### 4.2 Registering the App in `settings.py`

Django needs to be informed about the new app. Open `config/settings.py` in your text editor and locate `INSTALLED_APPS`.

Add `'students.apps.StudentsConfig'` (or simply `'students'`) to the list:

```python
# config/settings.py

INSTALLED_APPS = [
    # Built-in Django Apps
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # Third-Party Apps (Added here later)

    # Local Apps
    'students.apps.StudentsConfig',
]
```

---

## 05. Initial Database Setup & Running the Server

### 5.1 Applying Default Migrations

Django comes with default apps (Authentication, Admin, Sessions) that require initial database tables.

Run `migrate` to initialize the default SQLite database (`db.sqlite3`):

```bash
python manage.py migrate
```

*Expected Output Sample:*

```
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying sessions.0001_initial... OK
```

### 5.2 Starting the Development Server

Launch the built-in development server:

```bash
python manage.py runserver
```

*Expected Output:*

```
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
October 31, 2026 - 16:00:00
Django version 5.x.x, using settings 'config.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

Open your browser and navigate to `http://127.0.0.1:8000/`. You will see Django's default rocket welcome page!

---

## 06. Student Hands-On Practical Tasks

### Exercise 1: Verify Installed Apps & Status

1. Stop the server using `CTRL + C`.
2. Run system check validation:
   ```bash
   python manage.py check
   ```
3. Verify that `db.sqlite3` has been generated in your root directory:
   ```bash
   ls -la db.sqlite3
   ```

### Exercise 2: Binding Server to Specific Port

Practice running the development server on alternative ports:

```bash
# Run on port 8080
python manage.py runserver 8080

# Run accessible to local network requests
python manage.py runserver 0.0.0.0:8000
```

---

## 🚨 Troubleshooting & Common Errors

| Error Message | Cause | Resolution |
| :--- | :--- | :--- |
| `CommandError: 'config' conflicts with the name of an existing Python module` | Attempted to name project/app after standard Python modules. | Use a non-reserved directory name for your project/app. |
| `ModuleNotFoundError: No module named 'students'` | App not registered properly or directory path mismatch. | Check `INSTALLED_APPS` in `config/settings.py` for typos. |
| `That port is already in use.` | Another process is using port `8000`. | Stop the other process or specify another port (`python manage.py runserver 8080`). |
| `You have 18 unapplied migration(s)` | `runserver` executed before applying pending database changes. | Run `python manage.py migrate` in your terminal. |

---

## 📝 Lab Check-Off Checklist

Before leaving the lab, ensure you have completed:

* [ ] Created `config` project core using `django-admin startproject config .`.
* [ ] Created `students` app using `python manage.py startapp students`.
* [ ] Registered `students` in `config/settings.py`.
* [ ] Applied default migrations with `python manage.py migrate`.
* [ ] Successfully launched `python manage.py runserver` and viewed the welcome page in your browser.
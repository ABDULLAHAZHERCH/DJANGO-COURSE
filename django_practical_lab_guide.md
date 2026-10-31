# Practical Lab Guide 01: Introduction to Django & Environment Setup

## 🎯 Learning Objectives
By the end of this lab, you will be able to:
1. Understand the core concepts of Django and its **MVT (Model-View-Template)** architecture.
2. Differentiate between **MVC** and **MVT** patterns, and compare Django with PHP (Laravel), FastAPI, and Flask.
3. Configure a Linux development environment for Python web applications.
4. Create and manage isolated **Python Virtual Environments (`venv`)**.
5. Install, verify, and freeze dependencies for Django projects.

---

## 📑 Module Overview

| Section | Topic | Practical Focus |
|---|---|---|
| **01** | Theoretical Context & Architecture | MVT Flow & Framework Comparisons |
| **02** | System Prerequisites & Linux Setup | Terminal Navigation & Package Management |
| **03** | Python Environment Isolation | Setting up `python3-venv` |
| **04** | Installing & Verifying Django | Package installation via `pip` & validation |
| **05** | Hands-On Verification Tasks | Student Practice & Troubleshooting |

---

## 01. Theoretical Context: Django & MVT Architecture

### 1.1 What is Django?
Django is a high-level, open-source Python web framework designed to enable rapid development and clean, pragmatic design. It follows the **"Batteries Included"** philosophy—providing built-in tools for authentication, database ORM, administrative panel, forms, and security out-of-the-box.

### 1.2 The MVT Pattern Explained
Django uses the **MVT (Model-View-Template)** design pattern to separate concerns across an application.

```
+-----------------------------------------------------------------------+
|                           REQUEST FLOW                                |
|                                                                       |
|  [ Browser ] ---> ( Request ) ---> [ URL Router (urls.py) ]           |
|                                             |                         |
|                                             v                         |
|  [ Response ] <-- ( HTML ) <--- [ View (views.py) ]                   |
|                                     |        ^                        |
|                                     v        |                        |
|                                 [ Model ] [ Template ]                |
|                                (models.py) (HTML/DTL)                 |
+-----------------------------------------------------------------------+
```

1. **Model (`models.py`)**: Defines the data structures using Python classes. Each model maps directly to a single database table via Django's built-in Object-Relational Mapper (ORM).
2. **View (`views.py`)**: Contains the business logic. It handles HTTP requests, queries models for data, passes context data to templates, and returns HTTP responses.
3. **Template (`templates/*.html`)**: Handles the presentation layer using the Django Template Language (DTL) to dynamically generate HTML.

> 💡 **Key Takeaway:** Django's internal URL router (`urls.py`) acts as the **Controller** in traditional MVC architecture.

### 1.3 MVT vs. MVC Mapping

| MVC Term | Django Equivalent | Primary Responsibility |
|---|---|---|
| **Model** | Model (`models.py`) | Database schema definition, ORM queries, business rules. |
| **View** | Template (`templates/*.html`) | HTML generation, UI structure, dynamic display using DTL. |
| **Controller** | View (`views.py`) + URL Router (`urls.py`) | Request handling, routing, application logic execution. |

### 1.4 Web Framework Comparison Matrix

| Aspect | Django | PHP (Laravel) | FastAPI | Flask |
|---|---|---|---|---|
| **Language** | Python | PHP 8+ | Python | Python |
| **Style** | Full-Stack, MVT | Full-Stack, MVC | API-First, Micro | Microframework |
| **Database/ORM** | Built-in ORM | Eloquent ORM | External (SQLAlchemy/SQLModel) | External (SQLAlchemy) |
| **Admin Site** | Auto-generated | Extensions required | None | Extension (Flask-Admin) |
| **Async** | ASGI & WSGI support | Sync (FPM/Octane) | Native Async (ASGI) | Sync (WSGI default) |
| **Primary Use Case** | Complex web apps, CRUD, CMS | Standard web apps, legacy systems | High-speed REST APIs, ML APIs | Microservices, simple prototypes |

---

## 02. Linux System Environment Setup

Before building web applications, ensure your Linux workspace is updated and equipped with essential command-line tools.

### 2.1 Terminal Essentials Quick Reference

```bash
# Print working directory
pwd

# List contents with detailed permissions and hidden files
ls -la

# Create nested directories
mkdir -p ~/projects/student_management

# Navigate to directory
cd ~/projects/student_management

# Clear terminal screen
clear
```

### 2.2 Updating System Packages (Debian / Ubuntu)

Open your terminal and run the following commands:

```bash
# 1. Update local package index
sudo apt update

# 2. Upgrade existing system software
sudo apt upgrade -y

# 3. Install core development tools
sudo apt install -y git curl tree python3 python3-pip python3-venv
```

---

## 03. Setting Up a Python Virtual Environment

A **Virtual Environment** isolates project-specific dependencies to prevent version conflicts across different Python projects on the same machine.

### 3.1 Step-by-Step Virtual Environment Setup

#### Step 1: Create the Project Directory
```bash
mkdir -p ~/projects/student_management
cd ~/projects/student_management
```

#### Step 2: Initialize the Virtual Environment
We use the Python standard library module `venv` to create an isolated environment named `venv`.

```bash
python3 -m venv venv
```

* **Command Explanation:**
  * `python3`: Invokes the system's Python 3 interpreter.
  * `-m venv`: Executes the built-in `venv` module.
  * `venv`: Specifies the target folder name for the virtual environment binaries.

#### Step 3: Inspect the Generated Virtual Environment Structure
```bash
tree -L 2 venv/
```
*Expected Output:*
```text
venv/
├── bin/          # Contains Python binaries, pip, and activation scripts
├── include/      # C/C++ header files for compiled extensions
├── lib/          # Installed Python packages (site-packages)
└── pyvenv.cfg    # Environment configuration file
```

#### Step 4: Activate the Virtual Environment
```bash
source venv/bin/activate
```

Upon activation, your shell prompt will show `(venv)` at the beginning:
```bash
(venv) student@ubuntu:~/projects/student_management$
```

#### Step 5: Verify the Active Interpreter Path
```bash
which python
```
*Expected Output:*
```text
/home/student/projects/student_management/venv/bin/python
```

---

## 04. Installing and Managing Django

With the virtual environment active, install Django and verify its modules.

### 4.1 Installing Django via `pip`

```bash
# Upgrade pip to the latest version inside venv
python -m pip install --upgrade pip

# Install Django
pip install django
```

### 4.2 Verifying the Installation

Execute the following verification commands to ensure Django is properly installed and accessible:

```bash
# 1. Check Django version via CLI driver
django-admin --version

# 2. Check Django version via Python module execution
python -m django --version

# 3. Verify via programmatic inline Python script
python -c "import django; print(django.get_version())"

# 4. View package details
pip show django
```

*Expected Output Sample:*
```text
Name: Django
Version: 5.x.x
Summary: A high-level Python web framework that encourages rapid development and clean, pragmatic design.
Location: /home/student/projects/student_management/venv/lib/python3.x/site-packages
```

### 4.3 Freezing Dependencies into `requirements.txt`

To make your project reproducible for other developers or deployment servers, freeze the exact package dependencies into a manifest file:

```bash
pip freeze > requirements.txt
```

Inspect the generated `requirements.txt`:
```bash
cat requirements.txt
```

*Expected Content Sample:*
```text
asgiref==3.8.1
Django==5.1.2
sqlparse==0.5.1
```

---

## 05. Student Hands-On Practical Exercises

### Exercise 1: Directory & Permission Management
1. Inside `~/projects/student_management`, create a file named `setup_check.sh`.
2. Give the file executable permissions using octal notation (`755`).
3. Verify permissions using `ls -l`.

```bash
touch setup_check.sh
chmod 755 setup_check.sh
ls -l setup_check.sh
```

### Exercise 2: Virtual Environment Management Cycle
1. Deactivate your current virtual environment:
   ```bash
   deactivate
   ```
2. Verify that `which python` points back to system Python (`/usr/bin/python3`).
3. Reactivate the environment:
   ```bash
   source venv/bin/activate
   ```

---

## 🚨 Troubleshooting & Common Errors

| Error Message | Cause | Resolution |
|---|---|---|
| `externally-managed-environment` | Attempting `pip install` globally on modern Ubuntu/Debian. | Activate your virtual environment (`source venv/bin/activate`) before running `pip`. |
| `command not found: django-admin` | Running Django commands outside an active virtual environment. | Ensure `(venv)` is in your prompt. Re-run `source venv/bin/activate`. |
| `Unable to locate package python3-venv` | Outdated package manager index. | Run `sudo apt update` before installing. |
| `No module named django` | Django installed in a different environment or outside active `venv`. | Verify active Python path with `which python` and reinstall via `pip install django`. |

---

## 📝 Lab Check-Off Checklist
Before leaving the lab, ensure you have completed:
- [ ] Created `~/projects/student_management/` directory.
- [ ] Created and activated a Python `venv`.
- [ ] Installed Django inside the virtual environment.
- [ ] Verified installation via `python -m django --version`.
- [ ] Generated a valid `requirements.txt` file.
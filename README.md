# 🐍 Django & Web API Development: Practical Laboratory Guides

Welcome to the **Django & Web API Development Practical Laboratory Curriculum**. This repository contains a step-by-step, 9-part hands-on lab manual designed to guide students and developers from initial environment setup to building fully functional web applications and modern RESTful APIs using Python, Django, DRF, and FastAPI.

## 📚 Curriculum Structure & Lab Index

| Lab Guide | Topic | Core Concepts Covered | File Link | 
| ----- | ----- | ----- | ----- | 
| **Lab 01** | **Introduction & Environment Setup** | Django Architecture, MVT vs MVC, Framework Comparisons, Linux Terminal Setup, Python `venv` Isolation. | [`django_practical_lab_guide.md`](./django_practical_lab_guide.md) | 
| **Lab 02** | **Project & App Structure** | `django-admin startproject`, `startapp`, Architecture of `settings.py`, `urls.py`, `manage.py`, initial migrations, & `runserver`. | [`django_practical_lab_guide_02.md`](./django_practical_lab_guide_02.md) | 
| **Lab 03** | **Models & Django ORM** | Model class definitions, Field types, Constraints, `__str__()` method, `makemigrations`, `migrate`, and interactive Django shell CRUD queries. | [`django_practical_lab_guide_03.md`](./django_practical_lab_guide_03.md) | 
| **Lab 04** | **Views, Routing & DTL** | Function-Based Views (FBVs), App-level vs Project-level URL routing, Django Template Language (DTL) tags, variables, filters, & dynamic detail views. | [`django_practical_lab_guide_04.md`](./django_practical_lab_guide_04.md) | 
| **Lab 05** | **Forms & Request Processing** | HTTP `GET`/`POST` lifecycle, `forms.Form` vs `forms.ModelForm`, `{% csrf_token %}`, custom field clean validation, & the PRG (Post/Redirect/Get) pattern. | [`django_practical_lab_guide_05.md`](./django_practical_lab_guide_05.md) | 
| **Lab 06** | **Django Admin Customization** | Creating Superusers (`createsuperuser`), registering models, `ModelAdmin` customizations (`list_display`, `list_filter`, `search_fields`, `fieldsets`), & custom site headers. | [`django_practical_lab_guide_06.md`](./django_practical_lab_guide_06.md) | 
| **Lab 07** | **Authentication & Permissions** | User registration with `UserCreationForm`, `LoginView`, `LogoutView`, `@login_required` decorators, `@user_passes_test`, & authentication state template tags. | [`django_practical_lab_guide_07.md`](./django_practical_lab_guide_07.md) | 
| **Lab 08** | **Class-Based Views & Static/Media** | Generic CBVs (`ListView`, `DetailView`, `CreateView`, `UpdateView`, `DeleteView`), `LoginRequiredMixin`, static asset loading (`{% static %}`), Pillow, & `ImageField` uploads. | [`django_practical_lab_guide_08.md`](./django_practical_lab_guide_08.md) | 
| **Lab 09** | **Web APIs, DRF & FastAPI** | Native `JsonResponse`, REST principles, Django REST Framework (`ModelSerializer`, `ModelViewSet`), FastAPI microservice setup with Pydantic, & Swagger UI testing. | [`django_practical_lab_guide_09.md`](./django_practical_lab_guide_09.md) | 

## 🛠️ System Prerequisites

Before starting the practical exercises, ensure your development environment meets the following requirements:

* **Operating System:** Linux (Ubuntu 20.04/22.04 LTS recommended), macOS, or Windows WSL2.
* **Python:** Python `3.10+` installed.
* **Package Manager:** `pip` and `python3-venv` packages installed.
* **Code Editor:** VS Code, PyCharm, or any standard text editor.

## 🚀 Quick Start Guide

### 1. Clone the Repository

```bash
git clone https://github.com/ABDULLAHAZHERCH/DJANGO-COURSE.git
cd DJANGO-COURSE
```

### 2. Set Up a Python Virtual Environment

Follow **Lab 01** to configure your environment:

```bash
# Create environment
python3 -m venv venv

# Activate environment (Linux/macOS)
source venv/bin/activate

# Install Core Dependencies
pip install django djangorestframework fastapi "uvicorn[standard]" Pillow
```

## 📝 Key Features of these Lab Guides

Each laboratory manual in this repository follows a standardized, student-friendly structure:

* **🎯 Learning Objectives:** Clear outcomes mapped to each session.
* **📑 Module Overview:** High-level index of theory and practical tasks.
* **💡 Theoretical Context:** Architecture diagrams and framework concepts.
* **💻 Hands-on Step-by-Step Code Commands:** Terminal snippets with detailed explanation blocks.
* **🚨 Troubleshooting & Common Errors:** Practical solutions for real-world syntax and runtime errors.
* **📝 Lab Check-Off Checklist:** Verification checklists for instructors and students prior to lab sign-off.

## 🤝 Contribution & Usage

Feel free to fork this repository, submit pull requests, or adapt these lab guides for academic courses, workshops, and self-study modules.

*Created for Web Application Development & Software Engineering Lab Courses.*
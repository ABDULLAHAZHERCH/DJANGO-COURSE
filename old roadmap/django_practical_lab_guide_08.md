# Practical Lab Guide 08: Class-Based Views (CBVs) & Static/Media Asset Management

## 🎯 Learning Objectives

By the end of this lab, you will be able to:

1. Differentiate between **Function-Based Views (FBVs)** and **Class-Based Views (CBVs)**.

2. Implement Django's generic editing and display CBVs: `ListView`, `DetailView`, `CreateView`, `UpdateView`, and `DeleteView`.

3. Secure CBVs using `LoginRequiredMixin` and `PermissionRequiredMixin`.

4. Configure project settings for serving **Static Files** (CSS, JavaScript, logos).

5. Handle file uploads using `FileField` and `ImageField` in Django models.

6. Configure `MEDIA_URL` and `MEDIA_ROOT` to serve user-uploaded files during local development.

## 📑 Module Overview

| Section | Topic | Practical Focus | 
| ----- | ----- | ----- | 
| **01** | Introduction to Generic CBVs | Standardizing CRUD operations with class inheritance | 
| **02** | Refactoring Views to Generic CBVs | Implementing `ListView`, `DetailView`, `CreateView`, `UpdateView`, `DeleteView` | 
| **03** | Protecting CBVs with Mixins | Securing classes using `LoginRequiredMixin` | 
| **04** | Static Asset Management | `STATIC_URL`, `STATICFILES_DIRS`, and `{% static %}` | 
| **05** | Handling File & Image Uploads | `ImageField`, `MEDIA_ROOT`, and form `enctype="multipart/form-data"` | 
| **06** | Student Hands-On Exercises | Profile image uploading & CBV pagination | 
| **07** | Troubleshooting & Checkpoint | Common media upload and static file loading errors | 

## 01. Theoretical Context: Class-Based Views (CBVs)

While Function-Based Views (FBVs) provide complete control over request logic, **Class-Based Views (CBVs)** use Python class inheritance and mixins to eliminate boilerplate code for standard CRUD operations.

```
+-----------------------------------------------------------------------------------+
|                            GENERIC CBV HIERARCHY                                  |
|                                                                                   |
|                                  [ View ]                                         |
|                                     |                                             |
|             +-----------------------+-----------------------+                     |
|             |                                               |                     |
|     [ TemplateView ]                                [ ProcessFormView ]           |
|             |                                               |                     |
|    +--------+--------+                     +----------------+----------------+    |
|    |                 |                     |                |                |    |
| [ ListView ]   [ DetailView ]       [ CreateView ]   [ UpdateView ]   [ DeleteView ]
+-----------------------------------------------------------------------------------+

```

## 02. Implementing Generic CRUD CBVs (`students/views.py`)

Open `students/views.py` and implement generic CBVs for student records alongside access control mixins:

```
# students/views.py

from django.urls import reverse_lazy
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
from django.contrib.auth.mixins import LoginRequiredMixin
from .models import Student

class StudentListView(ListView):
    """
    Renders a list of Student records.
    Default template context variable: object_list (or student_list)
    Default template path: students/student_list.html
    """
    model = Student
    template_name = 'students/student_list.html'
    context_object_name = 'students'
    ordering = ['roll_no']
    paginate_by = 10  # Built-in pagination support!


class StudentDetailView(DetailView):
    """
    Renders details for a single Student instance by primary key or slug.
    Default template path: students/student_detail.html
    """
    model = Student
    template_name = 'students/student_detail.html'
    context_object_name = 'student'
    pk_url_kwarg = 'pk'


class StudentCreateView(LoginRequiredMixin, CreateView):
    """
    Renders and processes a form to create a new Student.
    LoginRequiredMixin ensures only authenticated users can access this view.
    """
    model = Student
    fields = ['name', 'roll_no', 'email', 'is_active']
    template_name = 'students/student_form.html'
    success_url = reverse_lazy('student_list_cbv')


class StudentUpdateView(LoginRequiredMixin, UpdateView):
    """
    Renders and processes a pre-populated form to edit an existing Student.
    """
    model = Student
    fields = ['name', 'email', 'is_active']
    template_name = 'students/student_form.html'
    success_url = reverse_lazy('student_list_cbv')


class StudentDeleteView(LoginRequiredMixin, DeleteView):
    """
    Renders a confirmation page and deletes the object upon POST submission.
    """
    model = Student
    template_name = 'students/student_confirm_delete.html'
    success_url = reverse_lazy('student_list_cbv')

```

### 2.1 Mapping CBVs in URL Patterns (`students/urls.py`)

To map CBVs in `urls.py`, call the class method `.as_view()`:

```
# students/urls.py

from django.urls import path
from .views import (
    StudentListView, 
    StudentDetailView, 
    StudentCreateView, 
    StudentUpdateView, 
    StudentDeleteView
)

urlpatterns = [
    path('cbv/', StudentListView.as_view(), name='student_list_cbv'),
    path('cbv/<int:pk>/', StudentDetailView.as_view(), name='student_detail_cbv'),
    path('cbv/add/', StudentCreateView.as_view(), name='student_create_cbv'),
    path('cbv/<int:pk>/edit/', StudentUpdateView.as_view(), name='student_update_cbv'),
    path('cbv/<int:pk>/delete/', StudentDeleteView.as_view(), name='student_delete_cbv'),
]

```

### 2.2 Creating Delete Confirmation Template (`student_confirm_delete.html`)

Create `students/templates/students/student_confirm_delete.html`:

```
touch students/templates/students/student_confirm_delete.html

```

Add deletion confirmation markup:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Confirm Delete</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; background-color: #f4f4f9; }
        .card { max-width: 450px; margin: auto; background: white; padding: 30px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .btn-danger { background-color: #e74c3c; color: white; padding: 10px 15px; border: none; border-radius: 4px; cursor: pointer; }
    </style>
</head>
<body>
    <div class="card">
        <h2>Delete Student Record?</h2>
        <p>Are you sure you want to permanently delete student <strong>"{{ object.name }}"</strong> (Roll No: {{ object.roll_no }})?</p>
        
        <form method="POST">
            {% csrf_token %}
            <button type="submit" class="btn-danger">Confirm Delete</button>
            <a href="{% url 'student_list_cbv' %}" style="margin-left: 10px;">Cancel</a>
        </form>
    </div>
</body>
</html>

```

## 03. Managing Static Assets (CSS, JS, Images)

Static files are non-dynamic assets (stylesheets, scripts, site logos) served directly by the web server.

### 3.1 Configuring Static Settings (`config/settings.py`)

Open `config/settings.py` and ensure the following settings are declared:

```
# config/settings.py

import os

STATIC_URL = '/static/'

# Additional locations of static files across the project
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
]

# Destination directory when running collectstatic for production deployment
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')

```

### 3.2 Creating the Project Static Directory

Create a global `static` folder in your project root:

```
mkdir -p static/css static/js static/images

```

Create a custom stylesheet `static/css/style.css`:

```
/* static/css/style.css */
body {
    background-color: #eef2f7;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}
.navbar {
    background-color: #2c3e50;
    color: white;
    padding: 15px 30px;
}

```

### 3.3 Loading Static Files in Templates

Load the `{% load static %}` tag at the top of your HTML templates:

```
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Student Directory</title>
    <!-- Linking project static CSS file -->
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
</head>
<body>
    <div class="navbar">
        <img src="{% static 'images/logo.png' %}" alt="Logo" height="30">
        <span>Student Management System</span>
    </div>
    ...
</body>
</html>

```

## 04. Handling File & Image Uploads (Media Assets)

Media files are files uploaded dynamically by users (e.g., student profile photos, PDF documents).

### 4.1 Installing Pillow Library

Django's `ImageField` requires the `Pillow` image processing library. Install it inside your virtual environment:

```
pip install Pillow
pip freeze > requirements.txt

```

### 4.2 Updating settings for Media Files (`config/settings.py`)

Open `config/settings.py` and append:

```
# config/settings.py

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

```

### 4.3 Updating Model with `ImageField` (`students/models.py`)

Open `students/models.py` and add a `profile_picture` field:

```
# students/models.py (Updated Student Model)

class Student(models.Model):
    name = models.CharField(max_length=100)
    roll_no = models.IntegerField(unique=True)
    email = models.EmailField(unique=True)
    is_active = models.BooleanField(default=True)
    profile_picture = models.ImageField(upload_to='student_photos/', blank=True, null=True)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.name} ({self.roll_no})"

```

Generate and apply the database migration:

```
python manage.py makemigrations students
python manage.py migrate

```

### 4.4 Serving Media Files in Development (`config/urls.py`)

Open `config/urls.py` and append static route helpers for local media file serving:

```
# config/urls.py

from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('students/', include('students.urls')),
]

# Serve media files in development environment
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)

```

### 4.5 Updating Forms and Templates for File Uploads

When submitting files via HTML forms, you **MUST** include `enctype="multipart/form-data"` on the `<form>` element.

#### Form HTML Update (`student_form.html`):

```
<form method="POST" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Save Student</button>
</form>

```

#### Displaying Uploaded Images in Template (`student_detail.html`):

```
{% if student.profile_picture %}
    <img src="{{ student.profile_picture.url }}" alt="{{ student.name }}" width="150" style="border-radius: 50%;">
{% else %}
    <p>No profile photo uploaded.</p>
{% endif %}

```

## 05. Student Hands-On Practical Tasks

### Exercise 1: Implement Pagination Controls

1. Open `students/templates/students/student_list.html`.

2. Add pagination links powered by `ListView`'s built-in `is_paginated` context:

   ```
   {% if is_paginated %}
     <div class="pagination" style="margin-top: 20px;">
       {% if page_obj.has_previous %}
         <a href="?page=1">&laquo; First</a>
         <a href="?page={{ page_obj.previous_page_number }}">Previous</a>
       {% endif %}

       <span>Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}</span>

       {% if page_obj.has_next %}
         <a href="?page={{ page_obj.next_page_number }}">Next</a>
         <a href="?page={{ page_obj.paginator.num_pages }}">Last &raquo;</a>
       {% endif %}
     </div>
   {% endif %}
   
   ```

## 🚨 Troubleshooting & Common Errors

| Error Message | Cause | Resolution | 
| ----- | ----- | ----- | 
| `ERRORS: Cannot use ImageField because Pillow is not installed.` | Missing dependency `Pillow`. | Run `pip install Pillow` inside your virtual environment. | 
| Uploaded image saves to database but link gives `404 Not Found` | Missing static route helper in `config/urls.py` or missing `MEDIA_ROOT` configuration. | Ensure `urlpatterns += static(settings.MEDIA_URL...)` is added to `config/urls.py`. | 
| Form submits without saving uploaded image file | Form tag is missing `enctype="multipart/form-data"`. | Add `enctype="multipart/form-data"` to the HTML `<form>` tag. | 

## 📝 Lab Check-Off Checklist

Before completing the course series, ensure you have verified:

* \[ \] Refactored CRUD views into generic CBVs (`ListView`, `DetailView`, `CreateView`, `UpdateView`, `DeleteView`).

* \[ \] Secured CBVs using `LoginRequiredMixin`.

* \[ \] Configured `STATIC_URL`, `STATICFILES_DIRS`, and loaded assets via `{% static %}`.

* \[ \] Installed `Pillow` and updated `Student` model with `ImageField`.

* \[ \] Configured `MEDIA_URL` and `MEDIA_ROOT` for uploaded assets.

* \[ \] Added `enctype="multipart/form-data"` to form templates and verified image uploads.
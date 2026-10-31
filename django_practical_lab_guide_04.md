# Practical Lab Guide 04: Views, URL Routing & Django Template Language (DTL)

## 🎯 Learning Objectives

By the end of this lab, you will be able to:

1. Understand the HTTP request-response lifecycle within Django's View layer.

2. Create function-based views (FBVs) in `views.py` that query the database via the ORM.

3. Configure modular URL routing using `include()` to connect app-level `urls.py` to project-level `config/urls.py`.

4. Structure template directories and configure template path discovery in `settings.py`.

5. Pass context dictionaries from views to dynamic HTML templates.

6. Use **Django Template Language (DTL)** features, including variable interpolation `{{ }}` and tags `{% %}` (`{% for %}`, `{% if %}`, `{% empty %}`).

## 📑 Module Overview

| Section | Topic | Practical Focus | 
| ----- | ----- | ----- | 
| **01** | The View & URL Architecture | Request-Response Cycle & URL Inclusion | 
| **02** | Creating Views (`views.py`) | Function-Based Views & Context Dictionaries | 
| **03** | Configuring URL Routes | App-level & Project-level `urls.py` | 
| **04** | HTML Templates & DTL | Folder structure & DTL Syntax (`for`, `if`, filters) | 
| **05** | Student Hands-On Exercises | Template extension & dynamic detail view | 
| **06** | Troubleshooting & Checkpoint | Common 404, TemplateDoesNotExist, & Syntax Errors | 

## 01. Theoretical Context: Views & URL Routing

In Django's **MVT architecture**, the **View** receives HTTP requests, interacts with the **Model** to retrieve data, and renders a **Template** to produce an HTTP response.

```
+-----------------------------------------------------------------------+
|                    REQUEST & VIEW ROUTING FLOW                        |
|                                                                       |
| [ Browser ] ---> HTTP Request ---> [ config/urls.py ]                 |
|                                            |                          |
|                                            v                          |
|                                   [ students/urls.py ]                |
|                                            |                          |
|                                            v                          |
| [ Browser ] <--- HTTP Response <--- [ views.student_list ]            |
|                                       /          \                    |
|                                      v            v                   |
|                                [ Student ORM ]  [ student_list.html ] |
+-----------------------------------------------------------------------+

```

## 02. Setting Up Views (`students/views.py`)

Open `students/views.py` and replace its contents with two view functions: a basic `HttpResponse` view and a rendered template view querying student records.

```
# students/views.py

from django.shortcuts import render
from django.http import HttpResponse
from .models import Student

def home_simple(request):
    """
    A simple view returning raw HTTP response text.
    """
    return HttpResponse("<h1>Welcome to the Student Management System</h1>")

def student_list(request):
    """
    Retrieves all student records and renders them using an HTML template.
    """
    # Fetch all records from database via ORM
    students = Student.objects.all().order_by('roll_no')
    
    # Construct context dictionary to pass data to template
    context = {
        'students': students,
        'page_title': 'Registered Students Directory',
        'total_count': students.count()
    }
    
    # Render template with context data
    return render(request, 'students/student_list.html', context)

```

## 03. Configuring URL Routing

Django uses modular URL routing. We define specific routes inside the app directory (`students/urls.py`) and include them in the root project config (`config/urls.py`).

### 3.1 Creating App-Level URLs (`students/urls.py`)

Create a new file `students/urls.py`:

```
touch students/urls.py

```

Add the following path configurations to `students/urls.py`:

```
# students/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path('simple/', views.home_simple, name='home_simple'),
    path('', views.student_list, name='student_list'),
]

```

### 3.2 Registering App URLs in Project Config (`config/urls.py`)

Open `config/urls.py` and include the `students` URL patterns using `django.urls.include`:

```
# config/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('students/', include('students.urls')), # Includes all routes from students app
]

```

*Resulting Route Map:*
* `http://127.0.0.1:8000/students/simple/` $\rightarrow$ `views.home_simple`
* `http://127.0.0.1:8000/students/` $\rightarrow$ `views.student_list`

## 04. Template Structure & Django Template Language (DTL)

### 4.1 Creating the Template Folder Hierarchy

Django looks for templates inside a `templates` directory within each app folder by default.

Create the required nested template directory:

```
mkdir -p students/templates/students

```

> 💡 **Best Practice:** Nesting `templates/students/` avoids naming collisions if multiple apps contain templates with identical names (e.g., `index.html`).

### 4.2 Creating the HTML Template (`student_list.html`)

Create `students/templates/students/student_list.html`:

```
touch students/templates/students/student_list.html

```

Add the following code containing DTL tags and variables:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ page_title }}</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; background-color: #f4f4f9; }
        h1 { color: #2c3e50; }
        .card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th, td { padding: 12px; text-align: left; border-bottom: 1px solid #ddd; }
        th { background-color: #3498db; color: white; }
        .badge-active { background-color: #2ecc71; color: white; padding: 4px 8px; border-radius: 4px; font-size: 0.8em; }
        .badge-inactive { background-color: #e74c3c; color: white; padding: 4px 8px; border-radius: 4px; font-size: 0.8em; }
    </style>
</head>
<body>
    <div class="card">
        <h1>{{ page_title }}</h1>
        <p>Total Registered Students: <strong>{{ total_count }}</strong></p>

        <table>
            <thead>
                <tr>
                    <th>Roll No</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Status</th>
                </tr>
            </thead>
            <tbody>
                {% for student in students %}
                <tr>
                    <td>{{ student.roll_no }}</td>
                    <td>{{ student.name|upper }}</td>
                    <td>{{ student.email }}</td>
                    <td>
                        {% if student.is_active %}
                            <span class="badge-active">Active</span>
                        {% else %}
                            <span class="badge-inactive">Inactive</span>
                        {% endif %}
                    </td>
                </tr>
                {% empty %}
                <tr>
                    <td colspan="4" style="text-align: center; color: #7f8c8d;">
                        No student records found in the database.
                    </td>
                </tr>
                {% endfor %}
            </tbody>
        </table>
    </div>
</body>
</html>

```

### 4.3 DTL Core Concepts Reference

| DTL Syntax | Type | Purpose | Example | 
| ----- | ----- | ----- | ----- | 
| `{{ variable }}` | Variable Interpolation | Renders value of Python variable or object property. | `{{ student.name }}` | 
| `{% tag %}` | Logic Tag | Controls template logic (loops, conditionals). | `{% for s in students %}` | 
| `{{ var\|filter }}` | Template Filter | Modifies variable display format inline. | `{{ student.name\|upper }}` | 
| `{% empty %}` | Clause inside `for` loop | Executes fallback HTML if iterable is empty/None. | Displays "No records" row | 

## 05. Student Hands-On Practical Tasks

### Exercise 1: Create a Dynamic Detail View

1. Open `students/views.py` and add a parameterized view function to retrieve a single student by `roll_no`:

   ```python
   from django.shortcuts import get_object_or_404

   def student_detail(request, roll_no):
       student = get_object_or_404(Student, roll_no=roll_no)
       return render(request, 'students/student_detail.html', {'student': student})
   ```

2. Add a dynamic URL pattern in `students/urls.py`:

   ```python
   path('<int:roll_no>/', views.student_detail, name='student_detail'),
   ```

3. Create `students/templates/students/student_detail.html` displaying individual profile details for `{{ student.name }}`, `{{ student.email }}`, and `{{ student.roll_no }}`.

4. Test in browser: Visit `http://127.0.0.1:8000/students/101/`.

## 🚨 Troubleshooting & Common Errors

| Error Message | Cause | Resolution | 
| ----- | ----- | ----- | 
| `TemplateDoesNotExist at /students/` | Django cannot find the target HTML file. | Ensure folder structure is `students/templates/students/student_list.html` and app is in `INSTALLED_APPS`. | 
| `Page not found (404)` | Request URL does not match patterns in `urls.py`. | Check `config/urls.py` inclusion prefix and `students/urls.py` paths for typos. | 
| `TemplateSyntaxError: Invalid block tag` | Typo in DTL tag syntax or missing `{% endfor %}` / `{% endif %}`. | Check template syntax for missing closing tags or missing percent `%` signs. | 
| `AttributeError: module 'students.views' has no attribute...` | URL pattern references a view function name that isn't defined in `views.py`. | Ensure the view function name in `urls.py` exactly matches `views.py`. | 

## 📝 Lab Check-Off Checklist

Before leaving the lab, ensure you have completed:

* \[ \] Written `home_simple` and `student_list` views in `students/views.py`.

* \[ \] Configured app-level routing in `students/urls.py`.

* \[ \] Included app routes inside `config/urls.py` using `include()`.

* \[ \] Created the template path `students/templates/students/student_list.html`.

* \[ \] Successfully rendered student data in browser using DTL tags (`{% for %}`, `{% if %}`).

* \[ \] Implemented the dynamic detail view exercise with `get_object_or_404`.
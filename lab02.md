# Lab 02: Django Templates, Layout Inheritance & Display Architecture

## 01. Introduction to Django Templates

In Django's **MVT (Model-View-Template)** architecture, the **Template layer** handles presentation. It decouples design from Python code by allowing developers to write static HTML markup embedded with dynamic placeholders and control statements using the **Django Template Language (DTL)**.

```text
+-------------------------------------------------------------------------------+
|                       TEMPLATE RENDERING ARCHITECTURE                         |
|                                                                               |
| [ View (views.py) ] --( Passes Context Dict )--> [ DTL Engine ]               |
|                                                         |                     |
| [ Template File (.html) ] --( Supplies Layout/DTL ) ----+                     |
|                                                         |                     |
|                                                         v                     |
| [ User Browser ] <------- ( Receives Rendered HTML ) ---+                     |
+-------------------------------------------------------------------------------+
```

### Why Use Templates Instead of Raw `HttpResponse` Strings?
1. **Separation of Concerns:** Web designers can update UI/HTML without touching backend Python logic.
2. **Reusability:** Common structural components (headers, navbars, footers) can be inherited across multiple pages.
3. **Security:** Built-in XSS (Cross-Site Scripting) protection through automatic HTML escaping.

---

## 02. Configuring Template Engines (`settings.py`)

Django handles template discovery and rendering via the `TEMPLATES` configuration dictionary in `config/settings.py`.

### Default Configuration in `config/settings.py`

```python
# config/settings.py

from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],  # Global project-level template directory
        'APP_DIRS': True,                  # Enables automatic searching in app/templates/
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

### Key Configuration Options Explained

| Parameter / Key | Type | Description |
| :--- | :--- | :--- |
| `'BACKEND'` | String | Dotted Python path to the template engine backend (`DjangoTemplates` or `Jinja2`). |
| `'DIRS'` | List of Paths | Additional filesystem directories searched before app subdirectories. Set to `[BASE_DIR / 'templates']`. |
| `'APP_DIRS'` | Boolean | When `True`, Django automatically searches inside a `templates/` folder within each registered app in `INSTALLED_APPS`. |
| `'context_processors'` | List of Callables | Python functions that automatically inject standard data (e.g., `request`, `user`, `messages`) into every template's context. |

---

## 03. Template Directory Structure & Namespacing

To avoid template name collisions across multiple apps, Django uses **App Namespacing**.

### Folder Layout Configuration

Create both a global template directory and an app-specific template directory:

```bash
cd ~/projects/student_management

# 1. Create project-level templates folder
mkdir -p templates

# 2. Create app-level namespaced templates folder
mkdir -p students/templates/students
```

### Recommended Project Tree

```text
student_management/
├── config/
│   └── settings.py
├── templates/                     # Project-level global templates (base, headers, error pages)
│   ├── base.html
│   ├── includes/
│   │   └── _navbar.html
│   └── 404.html
├── students/                      # App package
│   └── templates/
│       └── students/              # App namespace subdirectory
│           ├── student_list.html
│           └── student_detail.html
└── manage.py
```

> 💡 **Why Namespace App Templates?**  
> If two apps (`students` and `courses`) both have a template named `index.html`, Django's template loader will return the first one it finds in `INSTALLED_APPS`. Putting templates inside `students/templates/students/index.html` ensures you reference it uniquely as `'students/index.html'`.

---

## 04. Rendering Templates from Views (`render()`)

Django provides a shortcut function `render()` in `django.shortcuts` to load template files, inject data dictionaries, and return an `HttpResponse` object.

### Function Signature: `render()`

```python
render(request, template_name, context=None, content_type=None, status=None, using=None)
```

#### Argument Breakdown:
* **`request` (Required):** The `HttpRequest` object driving the current client request.
* **`template_name` (Required):** The relative path string of the target template (e.g., `'students/student_list.html'`).
* **`context` (Optional):** A Python dictionary mapping variable names to Python objects passed into the template. Defaults to an empty dict `{}`.
* **`status` (Optional):** HTTP status code for the response (e.g., `200`, `404`, `500`). Defaults to `200 OK`.

---

### Writing Views with Context Data (`students/views.py`)

Open `students/views.py` and replace its contents:

```python
# students/views.py

from django.shortcuts import render
from datetime import datetime

def student_list(request):
    """
    Renders a directory of student records using a Python context dictionary.
    """
    # Simulated database objects (Python dictionaries & primitives)
    student_records = [
        {
            'roll_no': 101,
            'name': 'Ali Khan',
            'email': 'ali@example.com',
            'gpa': 3.85,
            'is_enrolled': True,
            'joined_date': datetime(2023, 8, 15)
        },
        {
            'roll_no': 102,
            'name': 'Sara Ahmed',
            'email': 'sara@example.com',
            'gpa': 2.90,
            'is_enrolled': True,
            'joined_date': datetime(2024, 1, 10)
        },
        {
            'roll_no': 103,
            'name': 'Usman Raza',
            'email': 'usman@example.com',
            'gpa': 3.40,
            'is_enrolled': False,
            'joined_date': datetime(2022, 9, 1)
        }
    ]

    # Context dictionary passing data to the template
    context = {
        'page_title': 'Student Directory',
        'department': 'Computer Science',
        'students': student_records,
        'total_count': len(student_records),
        'server_timestamp': datetime.now()
    }

    return render(request, 'students/student_list.html', context)
```

---

## 05. Django Template Language (DTL) Syntax

The DTL syntax consists of four primary constructs:

| Construct | Syntax | Description | Example |
| :--- | :--- | :--- | :--- |
| **Variables** | `{{ variable }}` | Outputs a value from the context dictionary. | `{{ page_title }}` |
| **Tags** | `{% tag %}` | Executes control flow, loops, or layout commands. | `{% for s in students %}` |
| **Filters** | `{{ var\|filter }}` | Transforms/formats a variable's value inline. | `{{ student.name\|upper }}` |
| **Comments** | `{# comment #}` | Internal inline comments not sent to the browser HTML. | `{# Exclude inactive #}` |

---

### 5.1 Variable Resolution & Dot Lookup Rules

When Django evaluates `{{ student.name }}`, it uses **Dot Lookup** in the following order:

1. **Dictionary Lookup:** `student['name']`
2. **Attribute Lookup:** `student.name`
3. **Method Call:** `student.name()` (Calls method if callable and requires no arguments)
4. **List-Index Lookup:** `student[0]`

> ⚠️ **Missing Variables Rule:** If a variable does not exist in the context dictionary (e.g., `{{ missing_var }}`), Django renders it as an empty string `""` by default, rather than throwing a Python runtime exception.

---

### 5.2 Control Flow Tags

#### Conditional Statements (`{% if %}`, `{% elif %}`, `{% else %}`)

```html
{% if student.gpa >= 3.5 %}
    <span class="badge honor">Honor Roll</span>
{% elif student.gpa >= 2.0 %}
    <span class="badge passing">Passing</span>
{% else %}
    <span class="badge warning">Academic Probation</span>
{% endif %}
```

##### Supported Boolean Operators:
* Comparison: `==`, `!=`, `<`, `>`, `<=`, `>=`
* Logic: `and`, `or`, `not`
* Membership: `in`, `not in`

---

#### Iteration Loops (`{% for %}`, `{% empty %}`)

```html
<table>
    <thead>
        <tr>
            <th>#</th>
            <th>Roll No</th>
            <th>Name</th>
            <th>GPA</th>
            <th>Status</th>
        </tr>
    </thead>
    <tbody>
        {% for student in students %}
        <tr class="{% if forloop.counter|divisibleby:2 %}even-row{% else %}odd-row{% endif %}">
            <td>{{ forloop.counter }}</td>
            <td>{{ student.roll_no }}</td>
            <td>{{ student.name }}</td>
            <td>{{ student.gpa }}</td>
            <td>
                {% if student.is_enrolled %}
                    Active
                {% else %}
                    Inactive
                {% endif %}
            </td>
        </tr>
        {% empty %}
        <tr>
            <td colspan="5">No student records found in the database.</td>
        </tr>
        {% endfor %}
    </tbody>
</table>
```

##### Special `forloop` Variables Available Inside `{% for %}`:

| Variable | Description |
| :--- | :--- |
| `forloop.counter` | The 1-indexed counter of the loop iteration. |
| `forloop.counter0` | The 0-indexed counter of the loop iteration. |
| `forloop.revcounter` | The counter counting down to 1. |
| `forloop.first` | Boolean `True` if this is the first iteration. |
| `forloop.last` | Boolean `True` if this is the last iteration. |
| `forloop.parentloop` | Accesses the `forloop` object of a parent outer loop. |

---

### 5.3 Commonly Used Template Filters

Filters alter variable output and can be chained together using vertical bar `|` pipes.

```html
<!-- String formatting -->
<p>{{ student.name|lower }}</p>            <!-- Output: ali khan -->
<p>{{ student.name|upper }}</p>            <!-- Output: ALI KHAN -->
<p>{{ student.name|title }}</p>            <!-- Output: Ali Khan -->
<p>{{ "Hello World"|truncatechars:7 }}</p> <!-- Output: Hello... -->

<!-- Date formatting -->
<p>{{ student.joined_date|date:"F j, Y" }}</p> <!-- Output: August 15, 2023 -->
<p>{{ server_timestamp|date:"Y-m-d H:i" }}</p> <!-- Output: 2026-11-04 12:48 -->

<!-- Collection & Math utilities -->
<p>Total Count: {{ students|length }}</p>
<p>Default Value: {{ missing_bio|default:"No biography provided." }}</p>
<p>Chained Filters: {{ student.name|trim|upper }}</p>
```

---

### 5.4 HTML Auto-Escaping & Security

To prevent **XSS (Cross-Site Scripting)** attacks, Django automatically escapes HTML characters in dynamic output.

```python
# Context variable sent from views.py:
context = {'raw_script': "<script>alert('Hacked!');</script>"}
```

#### Rendering in HTML:
```html
<!-- Default behavior (Safely Escaped Output) -->
<div>{{ raw_script }}</div>
<!-- Browser output: &lt;script&gt;alert('Hacked!');&lt;/script&gt; -->

<!-- Disabling Auto-Escaping (Use only for trusted HTML content) -->
<div>{{ raw_script|safe }}</div>

<!-- Explicit Autoescape Tag Block -->
{% autoescape off %}
    <div>{{ raw_script }}</div>
{% endautoescape %}
```

---

## 06. Template Inheritance & Component Reusability

Template inheritance allows you to create a skeleton **base template** containing header, footer, and navigation elements, while child templates override specific content blocks.

```text
+-------------------------------------------------------------------------------+
|                             TEMPLATE INHERITANCE                              |
|                                                                               |
|  [ base.html ]                                                                |
|  +-------------------------------------------------------------------------+  |
|  | Header / Navigation Bar                                                 |  |
|  +-------------------------------------------------------------------------+  |
|  | {% block content %} ---> Overridden by Child ---> {% endblock %}        |  |
|  +-------------------------------------------------------------------------+  |
|  | Footer Copyright Section                                                 |  |
|  +-------------------------------------------------------------------------+  |
|                                     ^                                         |
|                                     |  {% extends 'base.html' %}              |
|  [ student_list.html ] -------------+                                         |
+-------------------------------------------------------------------------------+
```

---

### Step 6.1: Build Master Layout (`templates/base.html`)

Create `templates/base.html` in your global project directory:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Student System{% endblock %}</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; margin: 0; padding: 0; background-color: #f4f6f9; }
        header { background-color: #2c3e50; color: white; padding: 15px 30px; display: flex; justify-content: space-between; align-items: center; }
        header h2 { margin: 0; font-size: 1.5rem; }
        nav a { color: #ecf0f1; text-decoration: none; margin-left: 15px; font-weight: 500; }
        nav a:hover { text-decoration: underline; }
        main { max-width: 1000px; margin: 30px auto; background: white; padding: 30px; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
        footer { text-align: center; padding: 15px; background: #e2e8f0; color: #64748b; font-size: 0.9rem; position: fixed; bottom: 0; width: 100%; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th, td { border: 1px solid #cbd5e1; padding: 12px; text-align: left; }
        th { background-color: #3b82f6; color: white; }
        tr:nth-child(even) { background-color: #f8fafc; }
        .badge { padding: 4px 8px; border-radius: 4px; color: white; font-size: 0.8rem; font-weight: bold; }
        .badge.honor { background-color: #10b981; }
        .badge.passing { background-color: #3b82f6; }
        .badge.warning { background-color: #ef4444; }
    </style>
    {% block extra_css %}{% endblock %}
</head>
<body>

    <header>
        <h2>Student Management System</h2>
        <nav>
            <a href="{% url 'student_list' %}">Students</a>
            <a href="#">Courses</a>
            <a href="#">Help</a>
        </nav>
    </header>

    <main>
        <!-- Central dynamic content block -->
        {% block content %}
        {% endblock %}
    </main>

    <footer>
        <p>&copy; {% now "Y" %} Student Management System. All rights reserved.</p>
    </footer>

    {% block extra_js %}{% endblock %}
</body>
</html>
```

---

### Step 6.2: Component Partial Inclusion (`{% include %}`)

The `{% include %}` tag inserts a smaller HTML sub-template fragment into another template.

Create a partial file `templates/includes/_navbar.html`:

```html
<!-- templates/includes/_navbar.html -->
<div style="background-color: #1e293b; color: #94a3b8; padding: 8px 30px; font-size: 0.85rem;">
    <span>System Status: <strong>Operational</strong></span> | 
    <span>Department: <strong>{{ department|default:"General" }}</strong></span>
</div>
```

Include the partial inside `templates/base.html` directly below `<header>`:

```html
<header> ... </header>

{% include 'includes/_navbar.html' %}

<main> ... </main>
```

---

### Step 6.3: Extend Base Layout in App Template

Now construct `students/templates/students/student_list.html`:

```html
{% extends 'base.html' %}

{% block title %}Directory - {{ page_title }}{% endblock %}

{% block content %}
    <div style="display: flex; justify-content: space-between; align-items: center;">
        <h1>{{ page_title }}</h1>
        <p>Total Records: <strong>{{ total_count }}</strong></p>
    </div>

    <table>
        <thead>
            <tr>
                <th>#</th>
                <th>Roll No</th>
                <th>Full Name</th>
                <th>Email Address</th>
                <th>GPA</th>
                <th>Academic Status</th>
                <th>Registration Date</th>
            </tr>
        </thead>
        <tbody>
            {% for student in students %}
            <tr>
                <td>{{ forloop.counter }}</td>
                <td>{{ student.roll_no }}</td>
                <td><strong>{{ student.name|upper }}</strong></td>
                <td>{{ student.email }}</td>
                <td>{{ student.gpa|floatformat:2 }}</td>
                <td>
                    {% if student.gpa >= 3.5 %}
                        <span class="badge honor">Honor Roll</span>
                    {% elif student.gpa >= 2.0 %}
                        <span class="badge passing">Passing</span>
                    {% else %}
                        <span class="badge warning">Probation</span>
                    {% endif %}
                </td>
                <td>{{ student.joined_date|date:"M d, Y" }}</td>
            </tr>
            {% empty %}
            <tr>
                <td colspan="7" style="text-align: center;">No student records registered.</td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
{% endblock %}
```

---

## 07. Loading Static Files in Templates (`{% static %}`)

Static files (images, CSS files, JavaScript scripts) are rendered using the `{% load static %}` template tag.

### Step 7.1: Configure Static Settings (`config/settings.py`)

Ensure static paths are defined in `config/settings.py`:

```python
# config/settings.py

STATIC_URL = 'static/'

STATICFILES_DIRS = [
    BASE_DIR / 'static',
]
```

Create a global `static/` directory:

```bash
mkdir -p static/css static/js static/images
```

---

### Step 7.2: Reference Static Assets in HTML

```html
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Static Asset Demo</title>
    <!-- Linking external CSS static file -->
    <link rel="stylesheet" href="{% static 'css/custom.css' %}">
</head>
<body>
    <!-- Linking static image asset -->
    <img src="{% static 'images/logo.png' %}" alt="System Logo" width="120">

    <!-- Linking external JS static file -->
    <script src="{% static 'js/main.js' %}"></script>
</body>
</html>
```

---

## 08. Customizing HTTP Error Pages (404 & 500 Templates)

Django automatically looks for custom `404.html` and `500.html` files inside your global `templates/` directory when `DEBUG = False`.

### Step 8.1: Create Custom 404 Template (`templates/404.html`)

Create `templates/404.html`:

```html
{% extends 'base.html' %}

{% block title %}Page Not Found (404){% endblock %}

{% block content %}
    <div style="text-align: center; padding: 50px 20px;">
        <h1 style="font-size: 4rem; color: #ef4444; margin: 0;">404</h1>
        <h2>Page Not Found</h2>
        <p style="color: #64748b;">The requested resource URL was not found on this server.</p>
        <a href="{% url 'student_list' %}" style="display: inline-block; margin-top: 15px; padding: 10px 20px; background: #3b82f6; color: white; text-decoration: none; border-radius: 4px;">Return to Student List</a>
    </div>
{% endblock %}
```

---

### Step 8.2: Testing Custom Error Handler Views (`students/views.py`)

To test 404 handling during development:

Add a test view in `students/views.py`:

```python
# students/views.py

from django.http import Http404

def custom_404_test(request):
    """
    Explicitly raises an Http404 exception to test error templates.
    """
    raise Http404("Student Record Does Not Exist")
```

Add a path in `students/urls.py`:

```python
# students/urls.py

urlpatterns = [
    path('', views.student_list, name='student_list'),
    path('test-404/', views.custom_404_test, name='test_404'),
]
```

Temporarily set `DEBUG = False` and `ALLOWED_HOSTS = ['*']` in `config/settings.py` to view production error rendering:

```python
# config/settings.py (Testing ONLY)
DEBUG = False
ALLOWED_HOSTS = ['*']
```

Visit `http://127.0.0.1:8000/students/test-404/` to see your custom error layout.

---

## 09. Advanced Topic: Custom Template Tags & Filters

You can write custom Python functions to create your own filters and tags inside your Django app.

### Step 9.1: Directory Setup
Create a package named `templatetags` inside your app directory:

```bash
mkdir -p students/templatetags
touch students/templatetags/__init__.py
touch students/templatetags/student_extras.py
```

```text
students/
├── templatetags/
│   ├── __init__.py
│   └── student_extras.py
```

---

### Step 9.2: Create Custom Filter Function

Open `students/templatetags/student_extras.py`:

```python
# students/templatetags/student_extras.py

from django import template

register = template.Library()

@register.filter(name='grade_letter')
def grade_letter(gpa_value):
    """
    Converts numeric GPA into a letter grade string.
    Usage in template: {{ student.gpa|grade_letter }}
    """
    try:
        gpa = float(gpa_value)
        if gpa >= 3.7:
            return 'A'
        elif gpa >= 3.0:
            return 'B'
        elif gpa >= 2.0:
            return 'C'
        elif gpa >= 1.0:
            return 'D'
        else:
            return 'F'
    except (ValueError, TypeError):
        return 'N/A'
```

---

### Step 9.3: Load & Use Custom Filter in Template

```html
{% extends 'base.html' %}
{% load student_extras %}  <!-- Load custom filter package -->

{% block content %}
    <p>Numeric GPA: {{ student.gpa }}</p>
    <p>Letter Grade: <strong>{{ student.gpa|grade_letter }}</strong></p>
{% endblock %}
```

---

## 🚨 Troubleshooting & Common Errors

| Error Message | Cause | Resolution |
| :--- | :--- | :--- |
| `TemplateDoesNotExist at /students/` | Django cannot locate the requested HTML file. | Verify file path matches `students/templates/students/student_list.html` and `'students'` is in `INSTALLED_APPS`. |
| `TemplateSyntaxError: Invalid block tag` | Missing closing tag or invalid syntax (e.g., forgetting `{% endfor %}` or `{% endif %}`). | Inspect template tag syntax for missing `%` or missing closing tags. |
| `TemplateSyntaxError: 'static' is not a registered tag library` | Forgetting to load the static tag library at the top of the template file. | Add `{% load static %}` at the very top of your template. |
| Custom template filter not recognized | Django development server was not restarted after adding a new file in `templatetags/`. | Restart the server using `python manage.py runserver`. |
| Variable renders as empty string `""` | Misspelled key name in context dictionary passed from `views.py`. | Check dictionary keys in `views.py` match variable names in `{{ variable }}` exactly. |
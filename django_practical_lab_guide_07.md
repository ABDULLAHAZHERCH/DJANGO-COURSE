# Practical Lab Guide 07: User Authentication, Authorization & Permissions

## 🎯 Learning Objectives

By the end of this lab, you will be able to:

1. Understand Django's built-in authentication system (`django.contrib.auth`) and data model (`User`).

2. Create user registration forms and views using Django's built-in `UserCreationForm`.

3. Implement User Login and User Logout functionality using built-in or custom views.

4. Restrict view access using the `@login_required` decorator.

5. Dynamically alter HTML templates based on user authentication status (`user.is_authenticated`).

6. Manage user authorization, staff flags (`is_staff`), and permission groups.

## 📑 Module Overview

| Section | Topic | Practical Focus | 
| ----- | ----- | ----- | 
| **01** | Authentication vs. Authorization | Core security concepts & `django.contrib.auth` | 
| **02** | User Registration (`signup`) | Building registration logic with `UserCreationForm` | 
| **03** | User Login & Logout Views | Handling session creation and termination | 
| **04** | Access Control & `@login_required` | Restricting view execution to logged-in users | 
| **05** | Template Security & UI Adaptation | Conditional rendering with `{% if user.is_authenticated %}` | 
| **06** | Student Hands-On Tasks | Registration extending & Login redirect setup | 
| **07** | Troubleshooting & Checkpoint | Fixing common session and authentication errors | 

## 01. Theoretical Context: Authentication vs. Authorization

* **Authentication (AuthN):** Verifying **who** a user is (e.g., checking username and password credentials).

* **Authorization (AuthZ):** Determining **what** an authenticated user is permitted to do (e.g., editing records, accessing admin interfaces).

Django provides a complete, secure user management system out-of-the-box via `django.contrib.auth`.

```
+-----------------------------------------------------------------------+
|                    AUTHENTICATION LIFECYCLE                           |
|                                                                       |
|  [ User Input Credentials ] ---> [ authenticate(username, password) ] |
|                                                    |                  |
|                                              [ Credentials Valid? ]   |
|                                                    |                  |
|                                         +----------+----------+       |
|                                         |                     |       |
|                                       ( Yes )               ( No )    |
|                                         |                     |       |
|                                         v                     v       |
|                             [ login(request, user) ]   Render Error   |
|                             (Session Cookie Created)   Message        |
+-----------------------------------------------------------------------+

```

## 02. Building User Registration (`signup`)

We will use Django's built-in `UserCreationForm` to register new users safely with hashed passwords.

### 2.1 Writing the Registration View (`students/views.py`)

Open `students/views.py` and add the registration logic:

```
# students/views.py

from django.shortcuts import render, redirect
from django.contrib.auth.forms import UserCreationForm
from django.contrib import messages

def register(request):
    """
    Handles new user account registration using Django's UserCreationForm.
    """
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            user = form.save()
            username = form.cleaned_data.get('username')
            messages.success(request, f"Account created for {username}! You can now log in.")
            return redirect('login') # Redirect to login page after registration
        else:
            messages.error(request, "Registration failed. Please correct the errors below.")
    else:
        form = UserCreationForm()

    context = {
        'form': form,
        'page_title': 'User Account Registration'
    }
    return render(request, 'registration/register.html', context)

```

### 2.2 Registering the Route (`students/urls.py`)

Open `students/urls.py` and add the route for user registration:

```
# students/urls.py

from django.urls import path
from . import views

urlpatterns = [
    # ... existing routes ...
    path('register/', views.register, name='register'),
]

```

### 2.3 Creating the Registration Template (`register.html`)

Create a `registration` template directory inside `students/templates/`:

```
mkdir -p students/templates/registration
touch students/templates/registration/register.html

```

Add the following code to `register.html`:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ page_title }}</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; background-color: #f4f4f9; }
        .card { max-width: 450px; margin: auto; background: white; padding: 30px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .form-group { margin-bottom: 15px; }
        label { font-weight: bold; }
        input { width: 100%; padding: 8px; box-sizing: border-box; margin-top: 5px; }
        .btn { background-color: #3498db; color: white; padding: 10px; border: none; border-radius: 4px; width: 100%; cursor: pointer; }
        .helptext { display: block; color: #7f8c8d; font-size: 0.8em; margin-top: 4px; }
        .errorlist { color: #e74c3c; list-style: none; padding: 0; }
    </style>
</head>
<body>
    <div class="card">
        <h2>{{ page_title }}</h2>
        <form method="POST">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit" class="btn">Create Account</button>
        </form>
        <p>Already have an account? <a href="{% url 'login' %}">Log In here</a>.</p>
    </div>
</body>
</html>

```

## 03. User Login & Logout Implementation

Django provides built-in class-based authentication views (`LoginView` and `LogoutView`) inside `django.contrib.auth.views`.

### 3.1 Configuring Built-in Auth URLs (`config/urls.py`)

Open `config/urls.py` and attach Django's pre-built authentication paths:

```
# config/urls.py

from django.contrib import admin
from django.urls import path, include
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('students/', include('students.urls')),
    
    # Built-in Auth Routing
    path('login/', auth_views.LoginView.as_template_name('registration/login.html'), name='login'),
    path('logout/', auth_views.LogoutView.as_view(), name='logout'),
]

```

### 3.2 Setting Redirection URLs in `settings.py`

Open `config/settings.py` and specify default post-login and post-logout redirection targets at the bottom of the file:

```
# config/settings.py

# Redirection routes following Authentication events
LOGIN_REDIRECT_URL = '/students/'   # Redirect here after successful login
LOGOUT_REDIRECT_URL = '/login/'      # Redirect here after logging out
LOGIN_URL = '/login/'                # Redirect unauthenticated users here

```

### 3.3 Creating Login Template (`login.html`)

Create `students/templates/registration/login.html`:

```
touch students/templates/registration/login.html

```

Add the login form markup:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>User Login</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; background-color: #f4f4f9; }
        .card { max-width: 400px; margin: auto; background: white; padding: 30px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        input { width: 100%; padding: 8px; margin: 5px 0 15px 0; box-sizing: border-box; }
        .btn { background-color: #2ecc71; color: white; padding: 10px; border: none; border-radius: 4px; width: 100%; cursor: pointer; }
    </style>
</head>
<body>
    <div class="card">
        <h2>Sign In</h2>
        {% if form.errors %}
            <p style="color: #e74c3c;">Invalid username or password. Please try again.</p>
        {% endif %}

        <form method="POST">
            {% csrf_token %}
            <div>
                <label for="id_username">Username:</label>
                {{ form.username }}
            </div>
            <div>
                <label for="id_password">Password:</label>
                {{ form.password }}
            </div>
            <button type="submit" class="btn">Log In</button>
        </form>
        <p>Need an account? <a href="{% url 'register' %}">Register here</a>.</p>
    </div>
</body>
</html>

```

## 04. Access Control with `@login_required`

To prevent unauthenticated visitors from accessing specific views, wrap view functions with the `@login_required` decorator.

Open `students/views.py` and modify access permissions:

```
# students/views.py

from django.contrib.auth.decorators import login_required

# Public view: Anyone can see student lists
def student_list(request):
    students = Student.objects.all()
    return render(request, 'students/student_list.html', {'students': students})

# Protected View: Only logged-in users can add a student record
@login_required
def student_create(request):
    """
    Restricted to authenticated users. Unauthenticated visitors 
    will automatically be redirected to LOGIN_URL (/login/?next=/students/add/).
    """
    if request.method == 'POST':
        form = StudentForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('student_list')
    else:
        form = StudentForm()
    return render(request, 'students/student_form.html', {'form': form})

```

## 05. Adapting Templates Based on User Authentication

Django automatically injects the active `user` object into templates via request context. Use `user.is_authenticated` to show or hide UI elements.

Update `students/templates/students/student_list.html` navigation section:

```
<!-- Section inside student_list.html body -->
<nav style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px;">
    <div>
        {% if user.is_authenticated %}
            <span>Welcome, <strong>{{ user.username }}</strong>!</span>
            <a href="{% url 'student_create' %}" style="margin-left: 15px;">+ Add Student</a>
        {% else %}
            <span>Welcome, Guest!</span>
        {% endif %}
    </div>
    
    <div>
        {% if user.is_authenticated %}
            <!-- Logout POST form required in modern Django versions -->
            <form action="{% url 'logout' %}" method="POST" style="display: inline;">
                {% csrf_token %}
                <button type="submit" style="background: none; border: none; color: #e74c3c; cursor: pointer; text-decoration: underline;">
                    Log Out
                </button>
            </form>
        {% else %}
            <a href="{% url 'login' %}">Log In</a> | 
            <a href="{% url 'register' %}">Register</a>
        {% endif %}
    </div>
</nav>

```

## 06. Student Hands-On Practical Tasks

### Exercise 1: Staff-Only Authorization Check

1. Update `student_update` in `students/views.py` so only staff users (`user.is_staff`) can edit records.

2. Use Django's `user_passes_test` decorator:

   ```
   from django.contrib.auth.decorators import user_passes_test
   
   @login_required
   @user_passes_test(lambda u: u.is_staff)
   def student_update(request, roll_no):
       student = get_object_or_404(Student, roll_no=roll_no)
       form = StudentForm(request.POST or None, instance=student)
       if request.method == 'POST' and form.is_valid():
           form.save()
           return redirect('student_list')
       return render(request, 'students/student_form.html', {'form': form})
   
   ```

3. Test trying to edit a record as a standard user versus as a staff superuser.

## 🚨 Troubleshooting & Common Errors

| Error Message | Cause | Resolution | 
| ----- | ----- | ----- | 
| `Method Not Allowed (GET): /logout/` | Called HTTP `GET` on `LogoutView` in Django 5+. | Logout requests must be submitted via HTTP `POST` method containing `{% csrf_token %}`. | 
| Infinite redirect loop on protected page | `LOGIN_URL` is configured to point back to a `@login_required` view. | Ensure `LOGIN_URL = '/login/'` points to an un-decorated public login view. | 
| `TemplateDoesNotExist at /login/` | `LoginView` cannot find template at path `registration/login.html`. | Create template file inside `students/templates/registration/login.html`. | 

## 📝 Lab Check-Off Checklist

Before leaving the lab, ensure you have completed:

* \[ \] Built user registration view using `UserCreationForm`.

* \[ \] Configured `LoginView` and `LogoutView` routes.

* \[ \] Set `LOGIN_REDIRECT_URL` and `LOGOUT_REDIRECT_URL` in `settings.py`.

* \[ \] Protected the `student_create` view with the `@login_required` decorator.

* \[ \] Dynamically toggled UI login/logout links using `{% if user.is_authenticated %}`.

* \[ \] Tested restricted access control using `@user_passes_test`.
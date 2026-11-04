# Practical Lab Guide 06: Django Admin Interface & Customization

## 🎯 Learning Objectives

By the end of this lab, you will be able to:

1. Create administrative superusers using the `createsuperuser` CLI command.

2. Access and navigate the built-in Django Administrative Interface (`/admin/`).

3. Register custom models (`Student`) with the administrative site in `admin.py`.

4. Customize model representation using `ModelAdmin` options: `list_display`, `list_filter`, `search_fields`, `ordering`, and `readonly_fields`.

5. Add custom computed fields and fieldsets to organize the admin edit interface.

## 📑 Module Overview

| Section | Topic | Practical Focus | 
| ----- | ----- | ----- | 
| **01** | The Django Admin Philosophy | Automatic admin panel architecture | 
| **02** | Creating Superuser Credentials | `python manage.py createsuperuser` | 
| **03** | Registering Models (`admin.py`) | Basic vs. Advanced `ModelAdmin` registration | 
| **04** | Admin Site Customizations | Filters, search bars, list display, & ordering | 
| **05** | Advanced `ModelAdmin` Features | Fieldsets, read-only fields, & calculated list attributes | 
| **06** | Student Hands-On Tasks | Registration & customization exercises | 
| **07** | Troubleshooting & Checkpoint | Admin login issues & field lookup errors | 

## 01. Theoretical Context: Django Admin Interface

One of Django's most powerful features is its **automatic administrative interface**. It reads metadata from your `models.py` to generate a production-ready interface for managing content, site users, and application data.

```
+-----------------------------------------------------------------------+
|                    DJANGO ADMIN ARCHITECTURE                          |
|                                                                       |
|  [ models.py ]  ----( Metadata )----> [ admin.py ]                    |
|                                             |                         |
|                                             v                         |
|  [ Database ] <--- CRUD Operations <--- [ Django Admin Panel ]        |
|                                         (http://127.0.0.1:8000/admin) |
+-----------------------------------------------------------------------+

```

## 02. Creating a Superuser Account

To log into the Django admin interface, you must create an administrative user with full permissions (`is_superuser = True` and `is_staff = True`).

### 2.1 Executing `createsuperuser`

Ensure your virtual environment is active and execute:

```
python manage.py createsuperuser

```

### 2.2 Interactive Prompt Walkthrough

Follow the interactive terminal prompts:

```
Username (leave blank to use 'student'): admin
Email address: admin@example.com
Password: 
Password (again): 
Superuser created successfully.

```

> ⚠️ **Note:** When typing your password in the Linux terminal, characters will not appear on the screen for security reasons. Type your password and press `Enter`.

### 2.3 Accessing the Admin Panel

1. Start your local server if it is not already running:

   ```
   python manage.py runserver
   
   ```

2. Open your web browser and navigate to: `http://127.0.0.1:8000/admin/`

3. Log in using the superuser credentials created above.

## 03. Registering Models (`students/admin.py`)

By default, custom models are not visible in the admin dashboard until explicitly registered.

### 3.1 Simple Model Registration

Open `students/admin.py` and add:

```
# students/admin.py

from django.contrib import admin
from .models import Student

# Basic registration
admin.site.register(Student)

```

Refresh your browser at `http://127.0.0.1:8000/admin/`. The `Students` table will now appear under the `STUDENTS` application section.

## 04. Customizing the Admin Interface (`ModelAdmin`)

The default list view only displays the object’s `__str__()` representation. We can customize the interface using a custom `ModelAdmin` class and the `@admin.register` decorator.

Replace the contents of `students/admin.py` with:

```
# students/admin.py

from django.contrib import admin
from .models import Student

@admin.register(Student)
class StudentAdmin(admin.ModelAdmin):
    """
    Custom Admin Configuration for the Student model.
    """
    # Columns displayed in the list view table
    list_display = ('roll_no', 'name', 'email', 'is_active', 'created_at')

    # Clickable links in the list table to access object detail page
    list_display_links = ('roll_no', 'name')

    # Sidebar filter options
    list_filter = ('is_active', 'created_at')

    # Enable search functionality across these fields
    search_fields = ('name', 'roll_no', 'email')

    # Default ordering in admin table (- created_at means descending order)
    ordering = ('roll_no',)

    # Number of items per page
    list_per_page = 20

    # Inline editable fields directly from the list table view
    list_editable = ('is_active',)

```

### Key Customization Options Reference

| `ModelAdmin` Attribute | Purpose |
| ----- | ----- |
| `list_display` | Controls which model attributes appear as columns in the admin change list table. |
| `list_filter` | Adds a right sidebar containing dynamic filter controls (booleans, dates, foreign keys). |
| `search_fields` | Adds a search box that runs SQL `LIKE` / `icontains` queries over specified fields. |
| `list_editable` | Allows quick editing of specific fields directly from the list view table. |
| `list_display_links` | Specifies which column fields link to the model edit page. |

## 05. Advanced Layout Customization: Fieldsets & Calculated Columns

### 5.1 Organising Edit Forms using `fieldsets`

You can group related form fields into distinct visual sections on the change/edit page:

```
# students/admin.py (Updated StudentAdmin)

@admin.register(Student)
class StudentAdmin(admin.ModelAdmin):
    list_display = ('roll_no', 'name', 'email', 'is_active', 'created_at')
    search_fields = ('name', 'roll_no', 'email')
    list_filter = ('is_active',)

    # Non-editable fields on detail view
    readonly_fields = ('created_at',)

    # Group fields into visual sections on the detail/edit form
    fieldsets = (
        ('Personal Information', {
            'fields': ('name', 'email')
        }),
        ('Academic & Status Details', {
            'fields': ('roll_no', 'is_active')
        }),
        ('System Metadata', {
            'classes': ('collapse',), # Collapsible section
            'fields': ('created_at',),
        }),
    )

```

## 06. Student Hands-On Practical Tasks

### Exercise 1: Customize Admin Site Branding

Customize the global header text and title of the Django Admin interface.

Add the following configuration at the top of `students/admin.py`:

```
# students/admin.py

admin.site.site_header = "Student Management System Admin"
admin.site.site_title = "SMS Admin Portal"
admin.site.index_title = "Welcome to the Student Management Dashboard"

```

Refresh `http://127.0.0.1:8000/admin/` to view the updated site headers.

### Exercise 2: Add a Custom Calculated Admin Method

1. Add a custom method to `StudentAdmin` that displays whether a student's email is institutional:

   ```
   @admin.display(boolean=True, description='Is Institutional Email?')
   def is_official_email(self, obj):
       return obj.email.endswith('@university.edu')
   ```

2. Append `'is_official_email'` to `list_display`.

## 🚨 Troubleshooting & Common Errors

| Error Message | Cause | Resolution |
| ----- | ----- | ----- |
| `AlreadyRegistered: The model Student is already registered` | Model registered twice (e.g., using both `admin.site.register()` and `@admin.register`). | Remove one of the registration statements. |
| `FieldError: Unknown field(s)... specified for Student` | Typo in string entries inside `list_display`, `search_fields`, or `list_filter`. | Verify field names match the exact attributes in `students/models.py`. |
| `The value of 'list_editable[0]' refers to the first field in 'list_display'` | A field in `list_editable` is also set as a link in `list_display_links`. | Remove the editable field from `list_display_links`. |

## 📝 Lab Check-Off Checklist

Before leaving the lab, ensure you have completed:

* \[ \] Created a superuser via `python manage.py createsuperuser`.
* \[ \] Logged into the Django Admin interface at `/admin/`.
* \[ \] Registered the `Student` model using `@admin.register(Student)`.
* \[ \] Configured `list_display`, `search_fields`, and `list_filter`.
* \[ \] Organized the detail edit page using `fieldsets`.
* \[ \] Customized administrative headers via `admin.site.site_header`.
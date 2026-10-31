# Practical Lab Guide 03: Models, Database Migrations & Django ORM

## 🎯 Learning Objectives

By the end of this lab, you will be able to:

1. Create Python model classes in `models.py` that map to database tables.

2. Select appropriate Django field types (`CharField`, `IntegerField`, `EmailField`, `DateField`, `BooleanField`) and set field options (`unique`, `default`, `null`, `blank`, `max_length`).

3. Define the `__str__()` dunder method to customize string representations of model objects.

4. Generate and inspect migration files using `python manage.py makemigrations`.

5. Apply migrations to the database using `python manage.py migrate`.

6. Perform **CRUD (Create, Read, Update, Delete)** database operations via the Django Interactive Shell (`python manage.py shell`) using the Django ORM.

---

## 📑 Module Overview

| Section | Topic | Practical Focus |
| :--- | :--- | :--- |
| **01** | Understanding Django Models | Models as Python classes mapping to DB tables |
| **02** | Creating the `Student` Model | Defining fields, constraints, and `__str__` |
| **03** | Database Migrations Workflow | `makemigrations` and `migrate` commands |
| **04** | Django ORM & Interactive Shell | Executing CRUD queries in `python manage.py shell` |
| **05** | Student Hands-On Exercises | Model extension & ORM query challenges |
| **06** | Troubleshooting & Checkpoint | Fixing common ORM & Migration errors |

---

## 01. Conceptual Foundation: The Model Layer & ORM

### 1.1 What is Django ORM?
The **Object-Relational Mapper (ORM)** allows developers to interact with relational databases using **pure Python code** instead of writing raw SQL queries.

```
+-----------------------------------------------------------------------+
|                            DJANGO ORM                                 |
|                                                                       |
|   Python Model Class               Django ORM             SQL Table   |
|   class Student(Model)  <=======>  Python API  <=======>  CREATE      |
|     name = CharField()             .objects.all()         TABLE...    |
+-----------------------------------------------------------------------+
```

* **Python Class** $\rightarrow$ Database Table
* **Class Attributes** $\rightarrow$ Database Columns
* **Class Instances** $\rightarrow$ Database Rows

---

## 02. Defining Database Models

Open `students/models.py` in your text editor. We will construct a `Student` model to store student records.

### 2.1 Writing the `Student` Model

Replace the contents of `students/models.py` with the following code:

```python
# students/models.py

from django.db import models

class Student(models.Model):
    """
    Represents a student record in the Student Management System.
    """
    name = models.CharField(max_length=100)
    roll_no = models.IntegerField(unique=True)
    email = models.EmailField(unique=True)
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        """
        Returns a human-readable string representation of the object.
        Used in Django Admin, Shell, and Templates.
        """
        return f"{self.name} ({self.roll_no})"
```

### 2.2 Deep Dive: Field Types & Field Options

| Field Type / Option | Purpose | Description |
| :--- | :--- | :--- |
| `CharField(max_length=100)` | Text field | Variable-length string. Requires `max_length`. |
| `IntegerField()` | Numeric field | Stores 32-bit integer values. |
| `EmailField()` | String field | Validates that input matches an email format. |
| `BooleanField()` | Boolean field | Stores `True` or `False` (mapped to 1/0 or boolean in SQL). |
| `DateTimeField()` | Date & Time | `auto_now_add=True` sets timestamp automatically on creation. |
| `unique=True` | Constraint | Ensures no two records have the exact same value for this field. |
| `default=True` | Field Default | Assigns a fallback value if none is provided. |

---

## 03. Database Migrations Workflow

Migrations are Django’s way of propagating changes you make to your models (adding a field, deleting a model, etc.) into your database schema.

```
+-----------------------------------------------------------------------+
|                         MIGRATION LIFECYCLE                           |
|                                                                       |
|  1. Edit models.py  -->  2. makemigrations  -->  3. migrate           |
|  (Define Python class)   (Generate SQL instructions) (Execute SQL on DB) |
+-----------------------------------------------------------------------+
```

### Step 1: Generate Migration Files (`makemigrations`)

Run `makemigrations` to scan `models.py` for changes and construct a migration blueprint:

```bash
python manage.py makemigrations students
```

*Expected Output:*

```bash
Migrations for 'students':
  students/migrations/0001_initial.py
    - Create model Student
```

### Step 2: Inspect the Generated Migration File

Examine the generated file inside `students/migrations/0001_initial.py`:

```bash
cat students/migrations/0001_initial.py
```

### Step 3: Inspect Raw SQL (Optional Diagnostic Step)

To see the exact SQL commands Django will execute on SQLite/PostgreSQL, run `sqlmigrate`:

```bash
python manage.py sqlmigrate students 0001
```

*Expected Output Sample (SQLite Syntax):*

```sql
CREATE TABLE "students_student" (
    "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    "name" varchar(100) NOT NULL,
    "roll_no" integer NOT NULL UNIQUE,
    "email" varchar(254) NOT NULL UNIQUE,
    "is_active" bool NOT NULL,
    "created_at" datetime NOT NULL
);
```

### Step 4: Apply Migrations to Database (`migrate`)

Execute the migration file to alter the physical database schema:

```bash
python manage.py migrate
```

*Expected Output:*

```bash
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions, students
Running migrations:
  Applying students.0001_initial... OK
```

---

## 04. Interacting with Data via Django ORM (Interactive Shell)

The **Django Shell** loads your project configuration and models, allowing you to run Python ORM queries against your live database.

### 4.1 Launching Django Shell

```bash
python manage.py shell
```

*Prompt:*

```python
Python 3.x.x 
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> 
```

---

### 4.2 CRUD Operations with Django ORM

Execute the following commands line-by-line inside the interactive shell (`>>>`):

#### 1. Import the Model
```python
>>> from students.models import Student
```

#### 2. CREATE: Adding New Student Records

##### Method A: Instantiate and `.save()`
```python
>>> s1 = Student(name="Ali Khan", roll_no=101, email="ali@example.com")
>>> s1.save()  # Writes record to database
```

##### Method B: Using `.objects.create()` (Saves automatically)
```python
>>> s2 = Student.objects.create(name="Sara Ahmed", roll_no=102, email="sara@example.com")
>>> s3 = Student.objects.create(name="Usman Raza", roll_no=103, email="usman@example.com")
```

---

#### 3. READ: Querying the Database

##### Fetch All Records (`.all()`)
```python
>>> students = Student.objects.all()
>>> print(students)
<QuerySet [<Student: Ali Khan (101)>, <Student: Sara Ahmed (102)>, <Student: Usman Raza (103)>]>
```

##### Fetch Single Record by Primary Key/Lookup (`.get()`)
```python
>>> student_ali = Student.objects.get(roll_no=101)
>>> print(student_ali.name, student_ali.email)
Ali Khan ali@example.com
```

##### Filter Records (`.filter()`)
```python
>>> active_students = Student.objects.filter(is_active=True)
>>> print(active_students.count())
3
```

##### Filter using Lookup Field Modifiers (`__icontains`, `__gt`, etc.)
```python
>>> s_names = Student.objects.filter(name__icontains="sara")
>>> print(s_names)
<QuerySet [<Student: Sara Ahmed (102)>]>
```

---

#### 4. UPDATE: Modifying Records

##### Update single object:
```python
>>> student_sara = Student.objects.get(roll_no=102)
>>> student_sara.name = "Sara Ahmed Malik"
>>> student_sara.save()
```

##### Bulk Update via QuerySet:
```python
>>> Student.objects.filter(is_active=True).update(is_active=False)
3
```

---

#### 5. DELETE: Removing Records

```python
>>> student_to_delete = Student.objects.get(roll_no=103)
>>> student_to_delete.delete()
(1, {'students.Student': 1})
```

#### Exiting the Shell:
```python
>>> exit()
```

---

## 05. Student Hands-On Practical Tasks

### Exercise 1: Extend the Model

1. Re-open `students/models.py`.
2. Add a `gpa` field to the `Student` model using `models.FloatField(default=0.0)`.
3. Generate the migration: `python manage.py makemigrations`.
4. Apply the migration: `python manage.py migrate`.

### Exercise 2: Practice ORM Queries in Shell

1. Launch `python manage.py shell`.
2. Add 2 new students with GPAs `3.5` and `2.8`.
3. Write an ORM query to retrieve all students with a GPA greater than or equal to `3.0`:
   ```python
   Student.objects.filter(gpa__gte=3.0)
   ```

---

## 🚨 Troubleshooting & Common Errors

| Error Message | Cause | Resolution |
| :--- | :--- | :--- |
| `DoesNotExist: Student matching query does not exist.` | `.get()` query found zero matching records. | Use `.filter()` instead or wrap `.get()` in a `try...except Student.DoesNotExist` block. |
| `MultipleObjectsReturned: get() returned more than one Student` | `.get()` query found multiple records. | Use `.get()` only with unique fields like `id` or `roll_no`. Use `.filter()` for multiple results. |
| `IntegrityError: UNIQUE constraint failed: students_student.roll_no` | Attempted to insert a record with a duplicated `unique=True` field value. | Ensure unique data inputs for `roll_no` and `email`. |
| `Provide a one-off default` prompt during `makemigrations` | Added a non-nullable field to an existing model without a default value. | Select Option `1` to specify a temporary default value (e.g., `""` or `0`). |

---

## 📝 Lab Check-Off Checklist

Before leaving the lab, ensure you have completed:

* [ ] Created the `Student` model inside `students/models.py`.
* [ ] Defined the `__str__()` string representation method.
* [ ] Successfully executed `python manage.py makemigrations`.
* [ ] Applied migrations to the database with `python manage.py migrate`.
* [ ] Added, queried, updated, and deleted records via `python manage.py shell`.
* [ ] Extended the model with a `gpa` field and successfully migrated the database.
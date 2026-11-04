# Practical Lab Guide 09: Web APIs, Django REST Framework (DRF) & FastAPI

## 🎯 Learning Objectives

By the end of this lab, you will be able to:

1. Understand the core principles of **RESTful Web APIs** and JSON response structures.

2. Build simple JSON endpoints in native Django using `JsonResponse` and `serializers`.

3. Install and configure **Django REST Framework (DRF)**.

4. Create DRF `ModelSerializer` classes and `ModelViewSet` classes for rapid RESTful API development.

5. Set up a lightweight, high-performance standalone API using **FastAPI** and **Pydantic**.

6. Test API endpoints using interactive documentation interfaces (Swagger UI / OpenAPI).

## 📑 Module Overview

| Section | Topic | Practical Focus | 
| ----- | ----- | ----- | 
| **01** | Fundamentals of Web APIs & REST | Client-Server API Architecture & JSON data exchange | 
| **02** | Native Django JSON Endpoint | `JsonResponse` & `serializers.serialize()` | 
| **03** | Django REST Framework (DRF) | `ModelSerializer`, `@api_view`, & `ModelViewSet` | 
| **04** | Introduction to FastAPI | Asynchronous APIs, Type Hints, & Pydantic schemas | 
| **05** | Interactive API Testing | Exploring Swagger UI (`/docs`) & Redoc | 
| **06** | Student Hands-On Tasks | Build DRF filter endpoints & FastAPI POST routes | 
| **07** | Troubleshooting & Checkpoint | CORS errors, Serialization issues, & Pydantic errors | 

## 01. Theoretical Context: Web APIs & REST Architecture

An **Application Programming Interface (API)** allows decoupled systems (React/Vue frontends, mobile apps, or third-party backend microservices) to communicate over standard HTTP protocol using **JSON (JavaScript Object Notation)**.

```
+-----------------------------------------------------------------------------------+
|                            API REQUEST-RESPONSE CYCLE                             |
|                                                                                   |
|  [ React App / Mobile / Postman ]                                                 |
|               |                                                                   |
|               |---> HTTP GET /api/students/  -------------> [ Web API Endpoint ]  |
|               |                                                     |             |
|               |<--- 200 OK + JSON Payload <-------------------------+             |
|                                                                                   |
|  Sample JSON Payload:                                                             |
|  [{"id": 1, "name": "Ali Khan", "roll_no": 101, "is_active": true}]               |
+-----------------------------------------------------------------------------------+
```

### Standard HTTP Verbs in REST APIs

* **`GET`**: Retrieve resource(s).
* **`POST`**: Create a new resource.
* **`PUT` / `PATCH`**: Update an existing resource.
* **`DELETE`**: Remove a resource.

## 02. Native Django JSON Endpoints (`JsonResponse`)

Before using external libraries, let's observe how Django returns raw JSON payloads using built-in modules.

### 2.1 Writing a `JsonResponse` View (`students/views.py`)

Open `students/views.py` and append:

```python
# students/views.py

from django.http import JsonResponse
from django.core.serializers import serialize
import json
from .models import Student

def simple_student_api(request):
    """
    Native Django view returning JSON list of active students.
    """
    students = Student.objects.filter(is_active=True).values('id', 'name', 'roll_no', 'email')
    student_list = list(students) # Convert QuerySet to standard Python list
    
    return JsonResponse({
        'status': 'success',
        'count': len(student_list),
        'data': student_list
    })
```

### 2.2 Adding Endpoint URL (`students/urls.py`)

```python
# students/urls.py

from django.urls import path
from . import views

urlpatterns = [
    # ... existing routes ...
    path('api/simple/', views.simple_student_api, name='simple_student_api'),
]
```

Test in browser: Visit `http://127.0.0.1:8000/students/api/simple/`.

## 03. Building REST APIs with Django REST Framework (DRF)

While `JsonResponse` works for simple reads, **Django REST Framework (DRF)** provides robust tools like automatic serialization, input validation, authentication, and interactive API documentation.

### 3.1 Installing and Registering DRF

Ensure your virtual environment is active and install DRF:

```bash
pip install djangorestframework
pip freeze > requirements.txt
```

Register `'rest_framework'` in `config/settings.py`:

```python
# config/settings.py

INSTALLED_APPS = [
    # Built-in apps...
    'rest_framework',
    
    # Local apps...
    'students.apps.StudentsConfig',
]
```

### 3.2 Creating a Serializer (`students/serializers.py`)

A **Serializer** converts complex Django model instances into Python native datatypes (which can easily be rendered into JSON) and validates incoming JSON payloads during writes.

Create `students/serializers.py`:

```bash
touch students/serializers.py
```

Add the following code:

```python
# students/serializers.py

from rest_framework import serializers
from .models import Student

class StudentSerializer(serializers.ModelSerializer):
    """
    Serializes all fields of the Student model for REST API output and validation.
    """
    class Meta:
        model = Student
        fields = ['id', 'name', 'roll_no', 'email', 'is_active', 'created_at']
        read_only_fields = ['id', 'created_at']
```

### 3.3 Building a DRF ViewSet (`students/views.py`)

DRF **ViewSets** handle complete CRUD API functionality (`GET`, `POST`, `PUT`, `DELETE`) in a single concise class.

Append to `students/views.py`:

```python
# students/views.py

from rest_framework import viewsets
from .serializers import StudentSerializer

class StudentViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows Student records to be viewed, created, updated, or deleted.
    """
    queryset = Student.objects.all().order_by('roll_no')
    serializer_class = StudentSerializer
```

### 3.4 Registering DRF Router (`students/urls.py`)

DRF **Routers** automatically generate clean RESTful URL configurations for ViewSets.

Update `students/urls.py`:

```python
# students/urls.py

from django.urls import path, include
from rest_framework.routers import DefaultRouter
from . import views

# Create DRF Router and register ViewSets
router = DefaultRouter()
router.register(r'students', views.StudentViewSet, basename='student-api')

urlpatterns = [
    # Include DRF Router URLs
    path('api/v1/', include(router.urls)),
    
    # Simple JSON endpoint
    path('api/simple/', views.simple_student_api, name='simple_student_api'),
]
```

Test DRF Browsable API: Open `http://127.0.0.1:8000/students/api/v1/students/` in your browser.

## 04. Lightweight High-Performance APIs with FastAPI

**FastAPI** is a modern, fast (high-performance) web framework for building APIs with Python 3.8+ based on standard Python type hints and **Pydantic**. It is ideal for high-concurrency microservices.

### 4.1 Installing FastAPI & Uvicorn

Install FastAPI and its ASGI server **Uvicorn**:

```bash
pip install fastapi "uvicorn[standard]"
```

### 4.2 Creating a Standalone FastAPI Microservice

Create a standalone file `fastapi_app.py` in your project root:

```bash
touch fastapi_app.py
```

Add the following complete API code:

```python
# fastapi_app.py

from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel, EmailStr
from typing import List, Optional

# Initialize FastAPI app instance
app = FastAPI(
    title="Student Microservice API",
    description="High-performance API built with FastAPI and Pydantic",
    version="1.0.0"
)

# 1. Pydantic Schema Definitions (Data Validation)
class StudentSchema(BaseModel):
    id: int
    name: str
    roll_no: int
    email: EmailStr
    is_active: bool = True

class StudentCreateSchema(BaseModel):
    name: str
    roll_no: int
    email: EmailStr
    is_active: bool = True

# In-Memory Database for Demo
fake_student_db: List[StudentSchema] = [
    StudentSchema(id=1, name="Ali Khan", roll_no=101, email="ali@example.com", is_active=True),
    StudentSchema(id=2, name="Sara Ahmed", roll_no=102, email="sara@example.com", is_active=True),
]

# 2. API Endpoints
@app.get("/", tags=["Health Check"])
def read_root():
    return {"message": "FastAPI Service is Online"}

@app.get("/api/fast/students/", response_model=List[StudentSchema], tags=["Students"])
def get_all_students():
    """Retrieve list of all students."""
    return fake_student_db

@app.get("/api/fast/students/{student_id}", response_model=StudentSchema, tags=["Students"])
def get_student_by_id(student_id: int):
    """Retrieve single student by ID."""
    for student in fake_student_db:
        if student.id == student_id:
            return student
    raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Student not found")

@app.post("/api/fast/students/", response_model=StudentSchema, status_code=status.HTTP_201_CREATED, tags=["Students"])
def create_student(payload: StudentCreateSchema):
    """Create a new student entry."""
    new_id = len(fake_student_db) + 1
    new_student = StudentSchema(
        id=new_id,
        name=payload.name,
        roll_no=payload.roll_no,
        email=payload.email,
        is_active=payload.is_active
    )
    fake_student_db.append(new_student)
    return new_student
```

### 4.3 Running the FastAPI Application

Launch the ASGI Uvicorn server on port `8001` (to prevent conflicts with Django on `8000`):

```bash
uvicorn fastapi_app:app --reload --port 8001
```

*Terminal Output:*

```
INFO:     Started server process
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8001 (Press CTRL+C to quit)
```

## 05. Interactive API Documentation (Swagger UI)

One major advantage of FastAPI and DRF is auto-generated interactive documentation.

1. Open your browser to `http://127.0.0.1:8001/docs` to view **Swagger UI**.
2. Click on `POST /api/fast/students/` $\rightarrow$ Click **"Try it out"**.
3. Send a JSON body payload and test real-time validation provided by Pydantic.
4. Open `http://127.0.0.1:8001/redoc` to view alternative **ReDoc** documentation.

## 06. Student Hands-On Practical Tasks

### Exercise 1: DRF Search & Filtering

1. Install `django-filter`:
   ```bash
   pip install django-filter
   ```
2. Update `StudentViewSet` in `students/views.py` to enable search filtering:
   ```python
   from rest_framework import filters

   class StudentViewSet(viewsets.ModelViewSet):
       queryset = Student.objects.all()
       serializer_class = StudentSerializer
       filter_backends = [filters.SearchFilter]
       search_fields = ['name', 'roll_no', 'email']
   ```
3. Test searching in browser: `http://127.0.0.1:8000/students/api/v1/students/?search=Ali`

## 🚨 Troubleshooting & Common Errors

| Error Message | Cause | Resolution | 
| ----- | ----- | ----- | 
| `TypeError: Object of type Student is not JSON serializable` | Attempted to pass raw Django ORM QuerySet to `JsonResponse`. | Convert QuerySet values using `.values()` or pass through DRF Serializers. | 
| `Unprocessable Entity (422)` in FastAPI | Incoming JSON payload does not match Pydantic schema field types. | Check field data types (e.g., sending text for an integer `roll_no`). | 
| `Address already in use` when running Uvicorn | Port `8001` or `8000` is being used by another process. | Change port number (`uvicorn fastapi_app:app --port 8002`). | 

## 📝 Lab Check-Off Checklist

Before completing this lab, ensure you have verified:

* \[ \] Built a native `JsonResponse` endpoint in Django.
* \[ \] Installed `djangorestframework` and registered `'rest_framework'`.
* \[ \] Created `StudentSerializer` and `StudentViewSet`.
* \[ \] Registered DRF `DefaultRouter` in `students/urls.py`.
* \[ \] Installed `fastapi` and `uvicorn`.
* \[ \] Executed `fastapi_app.py` via Uvicorn on port `8001`.
* \[ \] Tested API endpoints via Swagger UI at `http://127.0.0.1:8001/docs`.
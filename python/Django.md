# Django

## Table Of Contents
- [Django Project vs Django App](#django-project-vs-django-app)
- [MTV Architecture](#mtv-architecture)
- [Request Flow in Django Deployment](#request-flow-in-django-deployment)
- [Web Server, Web Server Gateway Interface (WSGI) and Web Application](#web-server-web-server-gateway-interface-wsgi-and-web-application)
- [settings.py, urls.py, wsgi.py, asgi.py](#settingspy-urlspy-wsgipy-asgipy)
- [View (FBV vs CBV)](#view-fbv-vs-cbv)
- [Django URL Routing and Path Converters](#django-url-routing-and-path-converters)
- [Django Models & ORM (Fields, Relationships, Queries)](#django-models--orm-fields-relationships-queries)
- [Django Admin Interface](#django-admin-interface)
- [Migrations in Django](#migrations-in-django)
- [Template Engine(Filters, Inheritance, Context)](#template-enginefilters-inheritance-context)
- [Static Files vs Media Files](#static-files-vs-media-files)
- [render() vs redirect() vs HttpResponse](#render-vs-redirect-vs-httpresponse)
- [Built-in User Authentication System](#built-in-user-authentication-system)
- [Custom User Model & AbstractBaseUser](#custom-user-model--abstractbaseuser)
- [LoginRequiredMixin and Permission Decorators in Django](#loginrequiredmixin-and-permission-decorators-in-django)
- [Mixin in Django](#mixin-in-django)
- [Django Forms vs ModelForms](#django-forms-vs-modelforms)
- [Model Manager & QuerySet Customization](#model-manager--queryset-customization)
- [Middleware](#middleware)
- [Settings Management(DEBUG, ALLOWED_HOSTS, ENV handling)](#settings-managementdebug-allowed_hosts-env-handling)
- [CSRF Protection](#csrf-protection)
- [Transactions and atomic()](#transactions-and-atomic)


## Django Project vs Django App

n Django, a Project is the overall web application that contains the configuration and settings for the entire website. It acts as the central hub where different components of the application come together.

A Django App, on the other hand, is a modular component within the project that handles a specific functionality. For example, you might have separate apps for user authentication, blog management, or payment processing.

A single Django project can consist of multiple apps, and each app can be reused across different projects.

- The Project includes files like settings.py, urls.py, and wsgi.py which are responsible for configuration.

- An App includes files like models.py, views.py, admin.py, and apps.py, which define business logic and behavior for that specific feature.

`In Short:`
A Django Project is the container for your entire web application, while a Django App is a functional module that can be plugged into a project.


## MTV Architecture

Django follows the MVT architecture, which stands for Model - View - Template. It's a design pattern similar to MVC (Model-View-Controller), but tailored for web development in Django.

It separates concerns into three layers:

**`Model:`**
  Represents the database layer. It defines the structure of your data (tables), handles relationships, and provides a Pythonic way to interact with the database via Django ORM.

**`View:`**
  Contains the business logic. It processes HTTP requests, fetches data from models if needed, and returns a response — usually by rendering a template or returning JSON.

**`Template:`**
  The presentation layer. It’s an HTML file with Django’s templating syntax used to display dynamic data sent from the view.

## Request Flow in Django Deployment
1. Client (Browser) sends an HTTP request.

2. The request first reaches the **Web Server** (e.g., Nginx or Apache), which:
    - Handles SSL termination (HTTPS).
    - Serves static files directly (CSS, JS, images).
    - Acts as a reverse proxy, forwarding dynamic requests.

3. The Web Server forwards the dynamic request to the **WSGI Server** (e.g., Gunicorn or uWSGI).

4. The **WSGI Server** runs the Django application, processes the request using views and models, and generates a response.

5. The response flows back to the **WSGI Server**, then to the Web Server.

6. The **Web Server** sends the final HTTP response back to the Client.

```
Client
  ↓
[Nginx (Reverse Proxy)]
  ↓
[Gunicorn/uWSGI (WSGI Server)]
  ↓
[Django Application Code]
```


In production, a web server like Nginx receives client requests and handles static content and SSL. It forwards dynamic requests to a WSGI server like Gunicorn, which runs the Django app. The app processes the request and sends the response back through the WSGI server to the web server, which finally returns it to the client

## Web Server, Web Server Gateway Interface (WSGI) and Web Application

### Web Server
  - A web server is software that accepts HTTP requests from clients (like browsers) and sends back HTTP responses.
  
  - It mainly handles static content such as HTML files, CSS, JavaScript, images, etc.
  
  `Examples: Nginx, Apache, Microsoft IIS`
  
  - It can also act as a reverse proxy, forwarding requests to an application server for dynamic content.

### Web Server Gateway Interface (WSGI)

- WSGI is a standard Python interface between web servers and Python web applications or frameworks.

- It defines how a web server communicates with the web app to forward requests and receive responses.

- WSGI decouples web servers from Python applications, enabling any WSGI-compatible server to run any WSGI-compatible app.

- Popular WSGI servers: **`Gunicorn, uWSGI, mod_wsgi`**

## Web Application

- A web application is a software program that runs on a web server and provides dynamic content or services.

- In Python/Django context, the web app contains the business logic, database models, views, and templates.

- It processes incoming requests (via WSGI), executes application code, and returns HTTP responses.

- Example: A Django app that handles user authentication, CRUD operations, etc.


A web server like Nginx handles HTTP requests and static content, forwarding dynamic requests to a WSGI server such as Gunicorn that implements the WSGI interface to communicate with the web application (e.g., Django). The web app processes the request and returns a response, which flows back through the WSGI server and web server to the client.


## settings.py, urls.py, wsgi.py, asgi.py

### settings.py

This is the central configuration file for the Django project. It includes:

  - App registrations (INSTALLED_APPS)
  - Middleware configuration
  - Database setup
  - Static/media file paths
  - Security settings (CORS, CSRF, secret key)  
  - Logging, timezone, templates, and more

It acts as the backbone of the project where all global settings are maintained.

### urls.py

This file handles URL routing — it maps incoming requests to views.

  - Each app can have its own urls.py
  - The main project-level urls.py includes app-level routes using include()
  - It's the entry point for HTTP request routing

### wsgi.py (Web Server Gateway Interface)

Used for deploying synchronous Django apps.

WSGI is the traditional synchronous Python web server interface used to communicate between web servers and Python applications. It handles standard HTTP requests and responses.

```
import os
from django.core.wsgi import get_wsgi_application

# Set the settings module environment variable
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

# Get the WSGI application callable
application = get_wsgi_application()

# 'application' is the callable that the WSGI server (Gunicorn, uWSGI) calls for each request.
# Each request is handled synchronously in one thread per worker.
```

```
gunicorn myproject.wsgi:application --workers 4 --threads 2 --bind 0.0.0.0:8000

```

- --workers 4 → 4 worker processes (each a separate Python process)

- --threads 2 → each worker runs 2 threads to handle requests concurrently

- Total concurrency: 4 × 2 = 8 simultaneous requests



### asgi.py (Asynchronous Server Gateway Interface)
Used for handling asynchronous Django apps (like WebSockets, async views).

ASGI is its asynchronous, designed to support both synchronous and asynchronous communication, including WebSockets and long-lived connections, enabling modern real-time applications

```
import os
from django.core.asgi import get_asgi_application

# Set the settings module environment variable
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

# Get the ASGI application callable
application = get_asgi_application()

# 'application' is the async callable that the ASGI server (Uvicorn, Daphne) calls for each connection/request.
# It supports async concurrency, HTTP and WebSocket protocols.
```

```
uvicorn myproject.asgi:application --workers 4 --host 0.0.0.0 --port 8000

```

- --workers 4 → 4 worker processes

- Uvicorn doesn’t support threads by default, but it uses async to handle many requests within each worker



**In Simple**
- **settings.py** configures your Django project.
- **urls.py** maps URLs to views.
- **wsgi.py** provides an interface for WSGI servers to communicate with Django (synchronous).
- **asgi.py** provides an interface for ASGI servers to support asynchronous features like WebSockets

## View (FBV vs CBV)

In Django, a View is the part of the application that handles the request and returns a response. Views define what should happen when a user accesses a certain URL.

There are two types of views in Django:

### 1. Function-Based Views (FBV):

- These are simple Python functions.
- manually check the request method (GET, POST, etc.).
- Easy to read and good for simple logic.
  
**Use FBV when:**
- The logic is simple
- You need full control over the flow
- Readability is more important than reusability

  **Example**
```
  from django.http import HttpResponse

  def my_view(request):
    if request.method == 'GET':
        return HttpResponse("GET request")
    elif request.method == 'POST':
        return HttpResponse("POST request")
```

### 2. Class-Based Views (CBV):

- These are Python classes that use inheritance from Django’s view classes.
- Django handles HTTP methods with class methods like get(), post(), etc.
- CBVs offer built-in generic views for common patterns (like ListView, DetailView, etc.)
- Good for reusability, extensibility, and DRY (Don’t Repeat Yourself) principles.

**Use CBV when:**

- You have complex views
- You want to reuse logic or extend behavior
- You're building CRUD operations with less boilerplate



**Example**
```
from django.http import HttpResponse
from django.views import View

class MyView(View):
    def get(self, request):
        return HttpResponse("GET request")

    def post(self, request):
        return HttpResponse("POST request")

```

## Django URL Routing and Path Converters

In Django, URL routing is used to map incoming request URLs to the appropriate view functions or classes.

The mappings are defined in the urls.py file using path() or re_path() functions.

```
from django.urls import path
from . import views

urlpatterns = [
    path('hello/', views.hello_view),  # maps /hello/ to hello_view
]

```

### What are Path Converters?

Path converters allow you to capture dynamic parts of the URL and pass them as arguments to the view function.

They help in extracting values like integers, strings, slugs, UUIDs, etc., from the URL.

```
# urls.py
from django.urls import path
from .views import UserDetailView

urlpatterns = [
    path('user/<int:user_id>/', UserDetailView.as_view(), name='user-detail'),
]
```
```
# views.py
from django.http import JsonResponse
from django.views import View

class UserDetailView(View):
    def get(self, request, user_id):
        # You can use user_id to fetch user data
        return JsonResponse({"message": f"User ID is {user_id}"})
```

## Django Models & ORM (Fields, Relationships, Queries)

### Model:

- In Django, a Model is a Python class that defines the structure of your database table.
- Each model class maps to a table, and each attribute maps to a column.
- You define validations and constraints using Django’s built-in field types like CharField, IntegerField, DateField, etc.

**Example**
```
class Book(models.Model):
    title = models.CharField(max_length=100)
    published = models.DateField()

```

### Django ORM (Object Relational Mapper):

- Django ORM allows you to interact with the database using Python code instead of raw SQL.
- It maps Python objects to database tables, hence "Object Relational Mapper".
- You can create, read, update, and delete records using simple Python methods.

**Example**

```
# Create
Book.objects.create(title="Django Guide", published="2024-01-01")

# Read
book = Book.objects.get(id=1)

# Filter
books = Book.objects.filter(title__icontains="django")

# Update
book.title = "Updated Title"
book.save()

# Delete
book.delete()
```
### Model Relationships:
Django models support relationships between tables using:

| Relationship Type | Field Used        | Meaning                                 |
| ----------------- | ----------------- | --------------------------------------- |
| One-to-Many       | `ForeignKey`      | Many books can be written by one author |
| One-to-One        | `OneToOneField`   | One user has one profile                |
| Many-to-Many      | `ManyToManyField` | A student can take many courses         |


**Examples**

**One-to-Many (ForeignKey)**

```
class Author(models.Model):
    name = models.CharField(max_length=50)

class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)

```

**One-to-One (OneToOneField)**

```
class User(models.Model):
    name = models.CharField(max_length=100)

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField()

```

**Many-to-Many (ManyToManyField)**

```
class Course(models.Model):
    name = models.CharField(max_length=100)

class Student(models.Model):
    name = models.CharField(max_length=100)
    courses = models.ManyToManyField(Course)
```

In Django, Models define the structure of your database using Python classes. Django ORM lets you interact with those models using Python instead of SQL. You can define fields, relationships (like foreign keys), and perform queries easily using object methods like filter(), get(), create(), etc.


## Django Admin Interface

Django Admin is a powerful web UI to manage your database records. It’s auto-generated but highly customizable using ModelAdmin class. You can control what fields are shown, searchable, or filterable, and even manage related models inline.

Django provides a built-in admin panel that lets developers manage models (database tables) using a web interface — without writing any frontend code.

**To use the admin panel:**

1. Register your model in admin.py of the app.

```
from django.contrib import admin
from .models import Product

admin.site.register(Product)
```
2. Visit /admin in your browser and log in with your superuser account.

**Why Customize Admin Interface?**

The default admin is basic. We customize it to:

    - Show specific fields in list view
    - Add search and filters
    - Group fields in sections
    - Show related models inline

**Common Customizations**
```
from django.contrib import admin
from .models import Product

@admin.register(Product)
class ProductAdmin(admin.ModelAdmin):
    list_display = ('name', 'price', 'in_stock')  # columns in list view
    search_fields = ('name',)                     # search box
    list_filter = ('in_stock',)                   # right-side filters
    ordering = ('-created_at',)                   # order by latest

```
**Inline Admin for Related Models**
```
class CommentInline(admin.TabularInline):
    model = Comment

class BlogAdmin(admin.ModelAdmin):
    inlines = [CommentInline]
```

## Migrations in Django
Migrations in Django track changes to your models and sync them with the database.
makemigrations generates migration files, and migrate applies those changes to the actual DB.

**Two-Step Process:**

**makemigrations**

  &#8594; Command: python manage.py makemigrations
  
  &#8594; This scans your models.py files and creates migration files (Python code) that describe what changes should happen in the database.

**migrate**

  &#8594; Command: python manage.py migrate
  
  &#8594; This reads those migration files and executes the SQL queries to apply those changes to your database.

**Simple Analogy:**

 makemigrations `->` like drafting the plan (writes what should change)

 migrate `->` like executing the plan (applies changes in DB)

 **Examples**
```
python manage.py makemigrations
# Output: Created migration 0002_add_email_to_user

python manage.py migrate
# Output: Applying 0002_add_email_to_user... OK

```

```
python manage.py sqlmigrate yourapp 0002

```

## Template Engine(Filters, Inheritance, Context)

The template engine in Django allows you to dynamically generate HTML by mixing static HTML with dynamic data passed from the backend. It uses a syntax similar to Jinja.

Django Template Engine allows rendering dynamic HTML pages using context data, filters to format output, and template inheritance to keep layouts clean and reusable.

### Components

**1. Context**
- It's a dictionary of dynamic data that you pass from your view to the template.
- This data is used to render the page.

```
def my_view(request):
    context = {"name": "Naresh", "age": 30}
    return render(request, "profile.html", context)
```
```
<!-- profile.html -->
<h1>{{ name }}</h1>
<p>Age: {{ age }}</p>

```

**2. Template Filters**
- Filters are used to modify variables in templates.
- Syntax: {{ value|filter_name }}

**Common filters:**

- {{ name|lower }} → convert to lowercase
- {{ list|length }} → get length of a list
- {{ date|date:"Y-m-d" }} → format date

**3. Template Inheritance**

- Allows you to create a base layout and reuse it in other templates.
- Promotes DRY (Don't Repeat Yourself) principle.

```
<!-- base.html -->
<html>
  <head><title>{% block title %}My Site{% endblock %}</title></head>
  <body>
    <header>Header Section</header>
    {% block content %}{% endblock %}
    <footer>Footer</footer>
  </body>
</html>
```
```
<!-- home.html -->
{% extends "base.html" %}
{% block title %}Home Page{% endblock %}
{% block content %}
  <h1>Welcome Home!</h1>
{% endblock %}
```

## Static Files vs Media Files:

**Static Files**: Static files are assets like CSS, JavaScript, and images that do not change per user or request.

- Typically stored in each app’s static/ directory and collected via collectstatic.

```
# settings.py
STATIC_URL = '/static/'
STATICFILES_DIRS = [BASE_DIR / "static"]
```
```
{% load static %}
<link rel="stylesheet" href="{% static 'css/styles.css' %}">
<img src="{% static 'images/logo.png' %}">
```


**Media Files:** Media files are user-uploaded content such as profile pictures, documents, or attachments.

- Uploaded to MEDIA_ROOT, and served from MEDIA_URL.

```
# settings.py
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

```
<img src="{{ user.profile_picture.url }}">
```
```
# urls.py
from django.conf import settings
from django.conf.urls.static import static

urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)

```

## render() vs redirect() vs HttpResponse

### render():  
Combines a template with a context dictionary and returns an HttpResponse object with the rendered text.

```
from django.shortcuts import render

def home_view(request):
    return render(request, 'home.html', {'name': 'Naresh'})
```

### redirect():
Returns an HttpResponseRedirect to the specified URL or view name.

```
from django.shortcuts import redirect

def login_success(request):
    return redirect('dashboard')  # or use a URL path
```

### HttpResponse:
 Returns a custom raw HTTP response (plain text, JSON, HTML, XML, etc.).

```
from django.http import HttpResponse

def simple_response(request):
    return HttpResponse("Hello, this is raw response text")
```

**Summary**

- Use **render()** to serve HTML pages with dynamic data.

- Use **redirect()** to send users to a different URL after an action.

- Use **HttpResponse** when you need full control over the response, such as sending raw content or JSON (or for custom headers/status).

## Built-in User Authentication System

Django's built-in authentication system simplifies user management, offering secure login, logout, session control, and permission handling out of the box—ideal for both small and large applications.

**Core Features:**
- User registration and login/logout
- Password hashing and validation
- Session management
- User groups and permissions
- Authentication decorators/mixins (@login_required, LoginRequiredMixin)
- Admin integration for managing users

**Default User model includes:**

- username, password, email
- is_active, is_staff, is_superuser
- first_name, last_name

**Common Usage:**

```
from django.contrib.auth import authenticate, login, logout

# Authenticate user
user = authenticate(request, username='naresh', password='secret')
if user is not None:
    login(request, user)  # Log the user in
else:
    # Invalid credentials
    pass

# Logout
logout(request)
```

## Custom User Model & AbstractBaseUser
Django lets you fully customize the user model using AbstractBaseUser. This is useful when you want to store additional user info or change the login mechanism (like using email instead of username). You define your own fields, authentication rules, and manager logic.

**Default User model includes:**

- username, password, email
- is_active, is_staff, is_superuser
- first_name, last_name


**How to Do It?**

**Step 1: Inherit from AbstractBaseUser and PermissionsMixin**

```
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager, PermissionsMixin
from django.db import models

class CustomUserManager(BaseUserManager):
    def create_user(self, email, password=None, **extra_fields):
        if not email:
            raise ValueError("Email is required")
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)  # hashes password
        user.save(using=self._db)
        return user

    def create_superuser(self, email, password=None, **extra_fields):
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)
        return self.create_user(email, password, **extra_fields)

class CustomUser(AbstractBaseUser, PermissionsMixin):
    email = models.EmailField(unique=True)
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
    mobile_number = models.CharField(max_length=15, blank=True, null=True)

    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)

    objects = CustomUserManager()

    USERNAME_FIELD = 'email'  # login with email
    REQUIRED_FIELDS = ['first_name', 'last_name']  # prompted in createsuperuser
```
**Step 2: Update settings.py**

```
AUTH_USER_MODEL = 'yourapp.CustomUser'
```

**Step 3: Run migrations**

```
python manage.py makemigrations
python manage.py migrate
```

## LoginRequiredMixin and Permission Decorators in Django

Django provides both LoginRequiredMixin for class-based views and @login_required for function-based views to restrict access to authenticated users. For more fine-grained access control, @permission_required checks if the user has specific permissions. These tools are key for securing routes in Django applications.

**LoginRequiredMixin (used in Class-Based Views):**

- Ensures a user is logged in to access a Function-Based View.

```
from django.contrib.auth.decorators import login_required
from django.shortcuts import render

@login_required(login_url='/login/')
def profile_view(request):
    return render(request, 'profile.html')
```

**@permission_required decorator:**

- Checks whether the logged-in user has a specific permission (like app_label.permission_code).
  
```
from django.contrib.auth.decorators import permission_required

@permission_required('blog.add_post', raise_exception=True)
def add_post(request):
    # user must have 'add_post' permission in 'blog' app
    pass
```
- raise_exception=True returns HTTP 403 if permission is denied.

## Mixin in Django

A Mixin is a reusable class in Python or Django that provides additional functionality to other classes through multiple inheritance, without being a standalone class on its own.

- A mixin doesn't define a full view or class — it only "mixes in" extra behavior.

- Commonly used in Class-Based Views (CBVs) in Django to add modular, reusable logic like authentication, permissions, or logging.

```
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import TemplateView

class DashboardView(LoginRequiredMixin, TemplateView):
    template_name = "dashboard.html"

```

## Django Forms vs ModelForms 

**- A Django Form is a class that helps in creating and validating HTML forms in a clean and Pythonic way.**

```
from django import forms

class ContactForm(forms.Form):
    name = forms.CharField(max_length=100)
    email = forms.EmailField()
    message = forms.CharField(widget=forms.Textarea)
```

**- A ModelForm is a helper class that automatically creates a form based on a Django Model.**

```
from django.forms import ModelForm
from .models import Contact

class ContactModelForm(ModelForm):
    class Meta:
        model = Contact
        fields = ['name', 'email', 'message']
```

## Model Manager & QuerySet Customization

In Django, `objects` is the model manager, and .all() returns a `QuerySet`.
The manager (objects) is the interface to database operations, and the `QuerySet` is the actual collection of records.
We can customize the manager using `models.Manager` and customize the `QuerySet` using models.`QuerySet` to add reusable filters or logic.

**Example:**

```
from django.db import models
from django.utils import timezone

# Custom QuerySet
class PostQuerySet(models.QuerySet):
    def published(self):
        return self.filter(status='published')
    
    def draft(self):
        return self.filter(status='draft')

# Custom Manager
class PublishedManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(status='published')

# Post Model
class Post(models.Model):
    STATUS_CHOICES = (
        ('draft', 'Draft'),
        ('published', 'Published'),
    )

    title = models.CharField(max_length=200)
    slug = models.SlugField(unique_for_date='publish')
    body = models.TextField()
    publish = models.DateTimeField(default=timezone.now)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    status = models.CharField(max_length=10, choices=STATUS_CHOICES, default='draft')

    # Managers
    objects = models.Manager()  # Default manager
    published = PublishedManager()  # Custom manager
    custom_qs = PostQuerySet.as_manager()  # QuerySet-based manager

    class Meta:
        ordering = ['-publish']

    def __str__(self):
        return self.title

```

**Usage**
```
# In views.py or Django shell

# Get all posts
posts = Post.objects.all()

# Get all published posts
published_posts = Post.published.all()

# Get all draft posts
draft_posts = Post.custom_qs.draft()
```

**QuerySet methods inside a Manager**

To reuse custom QuerySet methods inside a custom Manager without using .as_manager(), override the Manager’s get_queryset() to return your custom QuerySet and delegate methods explicitly to it.

```
class PostQuerySet(models.QuerySet):
    def published(self):
        return self.filter(status='published')

class PublishedManager(models.Manager):
    def get_queryset(self):
        return PostQuerySet(self.model, using=self._db)

    def published(self):
        # Delegate to the queryset method
        return self.get_queryset().published()
```


**Summary**
A QuerySet defines how data is fetched, filtered, or modified. A Manager is the interface to call those methods on the model. By using as_manager(), we expose custom QuerySet logic through the manager, enabling clean and reusable queries.


## Django Signals

Django signals allow decoupled applications to get notified when certain events occur in the system. It's a way to execute side effects (like sending emails or creating related models) automatically when something happens—like saving or deleting a model instance.

The most common built-in signals are:

- `pre_save`: Triggered before a model's save() method is called.
- `post_save`: Triggered after a model's save() method is successfully called.
- `pre_delete`, `post_delete`, etc.

This promotes cleaner and modular code by separating logic from the model itself.

| Concept     | Description                                     |
| ----------- | ----------------------------------------------- |
| `Signal`    | The event (e.g., `post_save`, `pre_save`)       |
| `Receiver`  | The function that responds to the event         |
| `@receiver` | Decorator that links the signal to the function |
| `apps.py`   | Hooks the signal setup into Django app startup  |

**Example: Automatically Create a Profile When a User is Created**


**models.py**

```
from django.db import models
from django.contrib.auth.models import User

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField(blank=True)

```

**signals.py**
```
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User
from .models import Profile

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)

```

**apps.py**

```
from django.apps import AppConfig

class MyAppConfig(AppConfig):
    name = 'myapp'

    def ready(self):
        import myapp.signals

```

**Summary**

Django signals are a way to trigger actions automatically when model events happen, like creating a profile after a user registers using post_save. This keeps code clean, modular, and decoupled.


## Middleware

Middleware in Django is a framework-level hook that allows you to process requests and responses globally before they reach the view or after the response is returned. It's useful for tasks like authentication, logging, session management, and more.

- Django comes with built-in middleware like `AuthenticationMiddleware`, `SecurityMiddleware`, etc.
- Middleware components are chained together, and each middleware can modify the request or response.
- You can also create custom middleware to implement your own logic.


**Structure of Middleware (Very Basic)**

```
class MyMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response  # called once at startup

    def __call__(self, request):
        # Code before view (before the chef cooks)
        response = self.get_response(request)
        # Code after view (after food is served)
        return response

```

**Example 1: Timer Middleware (Logging how long a view takes)**

```
import time

class TimerMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        start = time.time()
        response = self.get_response(request)
        duration = time.time() - start
        print(f"{request.path} took {duration:.2f}s to process")
        return response

```

**Example 2: Block Specific IP**

```
from django.http import HttpResponseForbidden

class BlockIPMiddleware:
    BLOCKED_IPS = ['123.123.123.123']

    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        ip = request.META.get('REMOTE_ADDR')
        if ip in self.BLOCKED_IPS:
            return HttpResponseForbidden("Your IP is blocked.")
        return self.get_response(request)

```

**How to Activate**

Add to your Django settings.py:

```
MIDDLEWARE = [
    # other middleware...
    'myapp.middleware.TimerMiddleware',
    'myapp.middleware.BlockIPMiddleware',
]
```

## Settings Management(DEBUG, ALLOWED_HOSTS, ENV handling)

Django’s settings.py is the central configuration file for managing the behavior of your Django project. It contains everything from database configuration to middleware, installed apps, and security settings.

**DEBUG**

- Controls whether Django shows detailed error pages.
- Should be True only in development.
- Set to `False` in production to avoid exposing sensitive information.

**ALLOWED_HOSTS**

- A list of host/domain names that this Django site can serve.
- Required when DEBUG = False.
- Prevents HTTP Host header attacks.

`Example:`

```
ALLOWED_HOSTS = ['localhost', '127.0.0.1', 'yourdomain.com']
```

**Environment Variable Handling**

- Used to manage sensitive info like API keys, DB credentials, or secret keys.
- Recommended to use the os.environ module or third-party libraries like python-decouple or django-environ.

**Example using os.environ:**

```
import os

DEBUG = os.environ.get("DEBUG", "False") == "True"
SECRET_KEY = os.environ.get("SECRET_KEY", "your-default-key")
ALLOWED_HOSTS = os.environ.get("ALLOWED_HOSTS", "").split(",")

```

## CSRF Protection

CSRF (Cross-Site Request Forgery) is a security vulnerability where an attacker tricks a user’s browser into submitting unauthorized requests to a web application where the user is authenticated.

Django’s CSRF protection prevents this by requiring a token (a unique secret value) to be sent with any POST, PUT, DELETE, or PATCH requests that modify data. This token is verified on the server side to ensure the request is legitimate and originated from the trusted site.

**How Django Implements CSRF Protection:**
- Uses a middleware called CsrfViewMiddleware that checks incoming requests.
- For HTML forms, Django templates provide a {% csrf_token %} tag to include the token in forms.
- For AJAX requests, the token is included in headers or request data.
- If the token is missing or incorrect, Django raises a 403 Forbidden error.

**Disable CSRF Protection (Not Recommended for Production):**

You might disable CSRF protection temporarily or for specific views in some scenarios

**1. Disabling globally (not recommended):**

```
# In settings.py
MIDDLEWARE = [
    # Remove or comment out
    # 'django.middleware.csrf.CsrfViewMiddleware',
    # other middleware...
]

```

**2. Disabling for a specific view:**

```
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def my_view(request):
    # your view logic
```

## Transactions and atomic()

### Transaction

A transaction is a sequence of database operations that must be executed completely or not at all. It ensures data integrity by treating a group of operations as a single unit.

### Why Transactions?

`Without transactions:` If one DB operation fails, previous operations still persist, leads to inconsistent state.

`With transactions:` All operations succeed together, or none are applied if there's an error.

### atomic() in Django

Django provides transaction.atomic() to manage database transactions safely and easily.


```
from django.db import transaction

def create_order(request):
    with transaction.atomic():
        order = Order.objects.create(user=request.user)
        Payment.objects.create(order=order, amount=100)
        # If something goes wrong, both creations are rolled back

```

- If any exception occurs inside the atomic() block, all changes are rolled back automatically.


**Example with Exception:**

```
from django.db import transaction

def create_profile_and_wallet(user_data, wallet_data):
    try:
        with transaction.atomic():
            profile = Profile.objects.create(**user_data)
            Wallet.objects.create(profile=profile, **wallet_data)
            raise Exception("Unexpected error!")  # Will rollback both inserts
    except Exception as e:
        print("Error occurred:", e)

```

### Nested Transactions (Savepoints)

- atomic() also supports nested transactions via savepoints:

```
with transaction.atomic():
    # outer block
    with transaction.atomic():
        # inner block — can rollback separately

```
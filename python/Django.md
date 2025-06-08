# Django

## Table Of Contents
- [Django Project vs Django App](#django-project-vs-django-app)
- [MTV Architecture](#mtv-architecture)
- [Request Flow in Django Deployment](#request-flow-in-django-deployment)
- [Web Server, Web Server Gateway Interface (WSGI) and Web Application](#web-server-web-server-gateway-interface-wsgi-and-web-application)
- [settings.py, urls.py, wsgi.py, asgi.py](#settingspy-urlspy-wsgipy-asgipy)
- [View (FBV vs CBV)](#view-fbv-vs-cbv)


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


## 

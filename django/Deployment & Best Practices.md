# 10. Deployment & Best Practices

Moving from a local development environment to a live server is the final hurdle. This chapter covers the transition from "it works on my machine" to a secure, scalable, and production-ready application.

## The Production Stack

In development, you use `runserver` and SQLite. In production, you need a more robust stack to handle multiple users and high traffic.

* **Web Server (Nginx/Apache)**: Handles incoming traffic, manages SSL (HTTPS), and serves static files
* **WSGI/ASGI Server (Gunicorn/Uvicorn)**: The "translator" that sits between the web server and Django
* **Database (PostgreSQL/MySQL)**: A dedicated, high-performance database. SQLite is generally not recommended for production due to concurrency limits
* **Process Manager (Systemd/Docker)**: Ensures your application starts automatically and restarts if it crashes

### Recommended Production Stack

<details>

<summary>Show docker-compose example</summary>

```yaml
# docker-compose.yml (Docker approach)
version: '3.8'
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./static:/app/static
    depends_on:
      - app

  app:
    build: .
    command: gunicorn myproject.wsgi:application --bind 0.0.0.0:8000
    volumes:
      - .:/app
      - static:/app/static
    depends_on:
      - db
    environment:
        - DEBUG=False
        - DATABASE_URL=postgresql://user:password@db:5432/myapp

  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
```

</details>

## Deployment Checklist

Before you go live, you must change several settings in `settings.py` for security and performance.

## Mini Walkthrough: Minimal Production Deployment (Nginx + Gunicorn)

This walkthrough is a practical “first production deploy” path. It assumes you already have a server and a domain.

### 1. Switch off debug and set allowed hosts

```python
# settings.py
DEBUG = False
ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']
SECRET_KEY = os.environ.get('SECRET_KEY')
```

### 2. Configure static/media paths and collect static files

```python
# settings.py
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'

MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

```bash
python manage.py collectstatic
```

### 3. Run Django behind Gunicorn

```bash
gunicorn myproject.wsgi:application --bind 127.0.0.1:8000
```

### 4. Put Nginx in front (reverse proxy + static)

<details>

<summary>Show Nginx reverse proxy example</summary>

```nginx
server {
    server_name yourdomain.com www.yourdomain.com;

    location /static/ {
        alias /path/to/project/staticfiles/;
    }

    location /media/ {
        alias /path/to/project/media/;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

</details>

Once this works, you can add process management (Systemd), HTTPS certificates, monitoring, and CI/CD.

### 1. Security First

```python
# settings.py
DEBUG = False  # Most important setting for production
ALLOWED_HOSTS = ['www.yourdomain.com', 'yourdomain.com']
SECRET_KEY = os.environ.get('SECRET_KEY')  # Never hardcode this
```

### 2. Database Configuration

```python
# PostgreSQL example
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
        'OPTIONS': {
            'connect_timeout': 10,
        }
    }
}
```

### 3. Static and Media Files

```python
# settings.py
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# Additional static file finders
STATICFILES_FINDERS = [
    'django.contrib.staticfiles.finders.FileSystemFinder',
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
]
```

### 4. Security Settings

```python
# settings.py
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
X_FRAME_OPTIONS = 'DENY'

SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SESSION_COOKIE_HTTPONLY = True
CSRF_COOKIE_HTTPONLY = True
```

### 5. Email Configuration

```python
# settings.py
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = os.environ.get('EMAIL_HOST_USER')
EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_HOST_PASSWORD')
DEFAULT_FROM_EMAIL = 'noreply@yourdomain.com'
```

### 6. Logging Configuration

```python
# settings.py
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process} {message} {pathname} {lineno} {funcName}',
            'style': '{levelname} {asctime} {module} {process} {message} {pathname} {lineno} {funcName}',
        },
        'simple': {
            'format': '{levelname} {message}',
        },
    },
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': BASE_DIR / 'logs/django.log',
            'formatter': 'verbose',
        },
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'simple',
        },
        'mail_admins': {
            'level': 'ERROR',
            'class': 'django.utils.log.AdminEmailHandler',
            'formatter': 'verbose',
            'filters': ['require_debug_false'],
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file', 'console', 'mail_admins'],
            'level': 'INFO',
            'propagate': True,
        },
        'django.request': {
            'handlers': ['file', 'console'],
            'level': 'WARNING',
            'propagate': True,
        },
        'myapp': {
            'handlers': ['file', 'console'],
            'level': 'DEBUG',
            'propagate': True,
        },
    },
}
```

## Environment Variables

Use a library like `python-dotenv` or `django-environ` to keep sensitive information out of your version control (GitHub).

{% tabs %}
{% tab title=".env" %}
\`\`\`python

## .env file

DEBUG=False SECRET\_KEY=your-super-secret-key-here DATABASE\_URL=postgres://user:password@localhost:5432/dbname EMAIL\_HOST\_USER=your-email@gmail.com EMAIL\_HOST\_PASSWORD=your-app-password ALLOWED\_HOSTS=localhost,127.0.0.1,yourdomain.com `</div><div data-gb-custom-block data-tag="tab" data-title='settings.py'>`python

## settings.py

import os from pathlib import Path from dotenv import load\_dotenv

## Load environment variables

load\_dotenv()

BASE\_DIR = Path(**file**).resolve().parent.parent SECRET\_KEY = os.getenv('SECRET\_KEY') DEBUG = os.getenv('DEBUG', 'False') == 'True' ALLOWED\_HOSTS = os.getenv('ALLOWED\_HOSTS', 'localhost,127.0.0.1').split(',')

````</div></div>

## Static and Media Files

In production, Django does not serve static files. You must run:

```bash
python manage.py collectstatic
````

This gathers all assets into a single folder (`STATIC_ROOT`) so that Nginx can serve them directly.

#### Nginx Configuration

<details>

<summary>Show full Nginx config example</summary>

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;

    location /static/ {
        alias /path/to/your/project/staticfiles/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    location /media/ {
        alias /path/to/your/project/media/;
        expires 1y;
        add_header Cache-Control "public";
    }

    location / {
        proxy_pass http://unix:/run/gunicorn.sock;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

</details>

### Process Management

#### Systemd Service File

<details>

<summary>Show Systemd service example</summary>

```ini
# /etc/systemd/system/django.service
[Unit]
Description=Django Application
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/path/to/your/project
Environment=PATH=/path/to/venv/bin
ExecStart=/path/to/venv/bin/gunicorn \
    myproject.wsgi:application \
    --workers 3 \
    --bind unix:/run/gunicorn.sock

[Install]
WantedBy=multi-user.target

[Socket]
Listen=/run/gunicorn.sock
```

</details>

```bash
# Enable and start the service
sudo systemctl enable django
sudo systemctl start django
sudo systemctl status django
```

### Optional: Docker Deployment

<details>

<summary>Show Dockerfile example</summary>

```dockerfile
# Dockerfile
FROM python:3.11-slim

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Set work directory
WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy project
COPY . .

# Collect static files
RUN python manage.py collectstatic --noinput

# Expose port
EXPOSE 8000

# Run the application
CMD ["gunicorn", "myproject.wsgi:application", "--bind", "0.0.0.0:8000"]
```

</details>

### Monitoring and Logs

Once your app is live, you need to know if something goes wrong.

#### Logging

Configure Django's logging to write errors to a file or a service like Sentry.

<details>

<summary>Show LOGGING example</summary>

```python
# settings.py
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process} {message}',
        },
    },
    'handlers': {
        'file': {
            'level': 'ERROR',
            'class': 'logging.FileHandler',
            'filename': BASE_DIR / 'logs/django.log',
        },
        'sentry': {
            'level': 'ERROR',
            'class': 'sentry_sdk.integrations.django.DjangoIntegration',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file', 'sentry'],
            'level': 'INFO',
            'propagate': True,
        },
    },
}
```

</details>

#### Error Tracking

Tools like Sentry or New Relic provide real-time alerts when your users hit a 500 error.

<details>

<summary>Show Sentry example</summary>

```python
# settings.py
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration

sentry_sdk.init(
    dsn=os.environ.get('SENTRY_DSN'),
    integrations=[DjangoIntegration()],
    traces_sample_rate=1.0,
    send_default_pii=True,
    environment=os.environ.get('DJANGO_ENV', 'development'),
)
```

</details>

### Optional: Performance Optimization

#### Database Optimization

```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'myapp',
        'USER': 'myappuser',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': '5432',
        'OPTIONS': {
            'MAX_CONNS': 20,
            'CONN_MAX_AGE': 600,
        }
    }
}

# Connection pooling
DATABASE_POOL_ARGS = {
    'max_overflow': 10,
    'pool_size': 5,
    'recycle': 500,
    'pre_ping': True,
}
```

#### Caching

```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}

# Session caching
SESSION_ENGINE = 'django.contrib.sessions.backends.cache.CacheSession'
```

#### Application Performance

```python
# settings.py
USE_TZ = True
TIME_ZONE = 'UTC'

# Template caching
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'OPTIONS': {
            'loaders': [
                ('django.template.loaders.cached.Loader', [
                    'django.template.loaders.filesystem.Loader',
                    'django.template.loaders.app_directories.Loader',
                ]),
            ],
        },
    },
]
```

### Optional: Backup Strategy

#### Database Backups

<details>

<summary>Show database backup script</summary>

```bash
# PostgreSQL backup script
#!/bin/bash
BACKUP_DIR="/backups/postgres"
DATE=$(date +%Y-%m-%d_%H-%M-%S)
DB_NAME="myapp"
DB_USER="myappuser"

pg_dump -U $DB_USER -h localhost -F c $DB_NAME > "$BACKUP_DIR/$DB_NAME_$DATE.sql"
```

</details>

#### Media File Backups

<details>

<summary>Show media backup script</summary>

```bash
#!/bin/bash
BACKUP_DIR="/backups/media"
DATE=$(date +%Y-%m-%d_%H-%M-%S)
MEDIA_DIR="/path/to/your/project/media"

tar -czf "$BACKUP_DIR/media_$DATE.tar.gz" -C "$MEDIA_DIR" .
```

</details>

### Optional: Security Best Practices

#### Regular Maintenance

```python
# settings.py
# Password policies
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        'OPTIONS': {
            'min_length': 12,
        }
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
]

# Session security
SESSION_COOKIE_AGE = 86400  # 24 hours
SESSION_SAVE_EVERY_REQUEST = False  # Performance trade-off
```

#### Security Headers

```python
# settings.py
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_HSTS_SECONDS = 31536000  # 1 year
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
X_FRAME_OPTIONS = 'DENY'
```

### Optional: Testing in Production

#### Staging Environment

```python
# settings/staging.py
from .production import *

DEBUG = True
ALLOWED_HOSTS = ['staging.yourdomain.com']
```

#### Health Checks

<details>

<summary>Show health check example</summary>

```python
# health_check.py
from django.http import HttpResponse
from django.db import connection

def health_check(request):
    try:
        # Check database connection
        with connection.cursor() as cursor:
            cursor.execute("SELECT 1")

        # Check cache
        from django.core.cache import cache
        cache.set('health_check', 'ok', timeout=5)
        cache.get('health_check')

        return HttpResponse("OK", status=200)
    except Exception as e:
        return HttpResponse(f"Error: {str(e)}", status=500)
```

</details>

### Optional: CI/CD Pipeline

#### GitHub Actions Example

<details>

<summary>Show GitHub Actions example</summary>

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production
on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: |
            python -m pip install -r requirements.txt
      - name: Run tests
        run: |
            python manage.py test
      - name: Run migrations
        run: |
            python manage.py migrate --dry-run
      - name: Check static files
        run: |
            python manage.py collectstatic --dry-run
      - name: Run security checks
        run: |
            python manage.py check --deploy --dry-run
```

</details>

### Final Project Structure

After following all these chapters, your repository should look organized and professional:

```
root/
├── SUMMARY.md (The GitBook sidebar)
├── django.md (Introduction page)
├── .env (Secrets - ignored by git)
├── .gitignore (Includes .env, __pycache__, and db.sqlite3)
├── requirements.txt (Lists Django, DRF, Gunicorn, etc.)
└── django/                (Folder for all .md chapters)
    ├── Introduction to Web Frameworks.md
    ├── Routing.md
    ├── Views & Logic.md
    ├── Templates & Frontend Integration.md
    ├── Models & Databases.md
    ├── Forms.md
    ├── Authentication & Security.md
    ├── Django REST Framework.md
    ├── Intermediate Features.md
    └── Deployment & Best Practices.md
```

### Summary

* **Nginx and Gunicorn** are the industry standards for serving Django
* **PostgreSQL** is the preferred database for its reliability and features
* **Environment Variables** protect your secrets
* **DEBUG = False** is non-negotiable for live sites
* **Monitoring** is essential for production stability
* **Regular backups** protect against data loss
* **Performance optimization** ensures your app scales with traffic

### Important Keywords

#### **Production Environment**

Live server environment configured for real users, with security optimizations and performance settings.

#### **Web Server**

Server software (like Nginx or Apache) that handles HTTP requests and serves your application.

#### **WSGI (Web Server Gateway Interface)**

Python standard that connects web servers to Python web applications.

#### **ASGI (Asynchronous Server Gateway Interface)**

Modern asynchronous interface for Python web applications, supporting async/await.

#### **Gunicorn**

Popular WSGI server for Django applications in production.

#### **Uvicorn**

ASGI server for Django applications supporting async views.

#### **Process Manager**

System service that manages application lifecycle (start, restart, monitor).

#### **Systemd**

Linux init system and service manager used for process management.

#### **Docker**

Container platform that packages applications with all dependencies for deployment.

#### **Environment Variables**

Configuration values stored outside the codebase, typically in `.env` files.

#### **Collectstatic**

Django command that gathers all static files into a single location for serving.

#### **Static Files**

CSS, JavaScript, images, and other assets that don't change dynamically.

#### **Media Files**

User-uploaded files like images, documents, and other content.

#### **Logging**

Process of recording application events, errors, and performance metrics.

#### **Error Tracking**

Real-time monitoring system for application errors and exceptions.

#### **Sentry**

Popular error tracking service for Django applications.

#### **SSL/TLS**

Security protocol that encrypts data transmitted between users and your server.

#### **HTTPS**

Secure version of HTTP using SSL/TLS encryption.

#### **Database Migration**

Process of updating database schema to match model changes.

#### **Backup Strategy**

Plan for regularly backing up database and media files for disaster recovery.

#### **Health Check**

Endpoint or script that verifies application health and status.

#### **CI/CD Pipeline**

Automated testing and deployment process (Continuous Integration/Continuous Deployment).

#### **Load Balancing**

Distributing traffic across multiple server instances to handle high traffic.

#### **Scaling**

Process of increasing application capacity to handle more users or traffic.

#### **Performance Optimization**

Techniques to improve application speed and resource usage.

#### **Security Headers**

HTTP headers that enhance browser security and protect against attacks.

#### **Session Security**

Configuration options that protect user sessions from hijacking.

#### **Password Policies**

Rules and validation for user password strength and security.
{% endtab %}
{% endtabs %}

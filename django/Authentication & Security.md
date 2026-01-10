
# 7. Authentication & Security

Security and User Management are built into Django's core. Instead of building your own login system, you use Django's robust, pre-tested authentication framework to handle users, groups, and permissions.

## The Built-in User Model

Django provides a default User model that includes fields for:

- **Username and Password** (stored as a secure hash)
- **Email address**
- **First and Last name**
- **Flags**: `is_staff` (access to admin), `is_active`, and `is_superuser`

### Using the User Model

```python
from django.contrib.auth.models import User

# Create a new user
user = User.objects.create_user(
    username='john_doe',
    email='john@example.com',
    password='secure_password123'
)

# Check authentication
from django.contrib.auth import authenticate
user = authenticate(username='john_doe', password='secure_password123')

# Login user
from django.contrib.auth import login
login(request, user)
```

### Custom User Model

For production applications, it's recommended to create a custom user model:

```python
# models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    phone_number = models.CharField(max_length=15, blank=True)
    birth_date = models.DateField(null=True, blank=True)

    def __str__(self):
        return self.username

# settings.py
AUTH_USER_MODEL = 'myapp.CustomUser'
```

## Authentication Views

Django includes ready-to-use views for common tasks like logging in, logging out, and password management.

### Login and Logout

You don't need to write the logic for these; you simply point your `urls.py` to Django's built-in views.

```python
# urls.py
from django.contrib.auth import views as auth_views
from django.urls import path

urlpatterns = [
    # Uses 'registration/login.html' by default
    path('login/', auth_views.LoginView.as_view(), name='login'),
    path('logout/', auth_views.LogoutView.as_view(), name='logout'),

    # Password management
    path('password_change/', auth_views.PasswordChangeView.as_view(), 
         name='password_change'),
    path('password_change/done/', auth_views.PasswordChangeDoneView.as_view(), 
         name='password_change_done'),
    path('password_reset/', auth_views.PasswordResetView.as_view(), 
         name='password_reset'),
    path('password_reset/done/', auth_views.PasswordResetDoneView.as_view(), 
         name='password_reset_done'),
    path('reset/<uidb64>/<token>/', auth_views.PasswordResetConfirmView.as_view(), 
         name='password_reset_confirm'),
    path('reset/done/', auth_views.PasswordResetCompleteView.as_view(), 
         name='password_reset_complete'),
]
```

### Custom Login Template

```html
<!-- templates/registration/login.html -->
{% extends 'base.html' %}

{% block title %}Login{% endblock %}

{% block content %}
<div class="login-container">
    <h2>Login</h2>

    {% if form.errors %}
        <div class="alert alert-danger">
            {% for field, errors in form.errors.items %}
                {% for error in errors %}
                    <p>{{ error }}</p>
                {% endfor %}
            {% endfor %}
        </div>
    {% endif %}

    <form method="post">
        {% csrf_token %}

        <div class="form-group">
            <label for="{{ form.username.id_for_label }}">Username:</label>
            {{ form.username }}
        </div>

        <div class="form-group">
            <label for="{{ form.password.id_for_label }}">Password:</label>
            {{ form.password }}
        </div>

        <button type="submit" class="btn btn-primary">Login</button>
    </form>

    <p>
        <a href="{% url 'password_reset' %}">Forgot your password?</a>
    </p>
</div>
{% endblock %}
```

## Mini Walkthrough: Protect a Page (Login → Redirect → Template Link)

This walkthrough shows a common pattern: require login for a view, then provide a login link in your base template.

{% tabs %}

{% tab title="views.py" %}

```python
from django.contrib.auth.decorators import login_required
from django.shortcuts import render

@login_required
def dashboard(request):
    return render(request, 'dashboard.html')
```

{% endtab %}

{% tab title="urls.py" %}

```python
from django.contrib.auth import views as auth_views
from django.urls import path

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(), name='login'),
    path('logout/', auth_views.LogoutView.as_view(), name='logout'),
]
```

{% endtab %}

{% tab title="base.html" %}

```html
<!-- templates/base.html -->
{% if user.is_authenticated %}
  <a href="{% url 'logout' %}">Logout</a>
{% else %}
  <a href="{% url 'login' %}">Login</a>
{% endif %}
```

{% endtab %}

{% endtabs %}

## Checking Authentication in Views

You can check if a user is logged in directly within your logic using the request object.

```python
def my_view(request):
    if request.user.is_authenticated:
        # Show private data
        return render(request, 'private_data.html')
    else:
        # Redirect to login
        return redirect('login')

# Access user properties
def profile_view(request):
    if request.user.is_authenticated:
        context = {
            'username': request.user.username,
            'email': request.user.email,
            'first_name': request.user.first_name,
            'last_name': request.user.last_name,
            'is_staff': request.user.is_staff,
        }
        return render(request, 'profile.html', context)
    else:
        return redirect('login')
```

## Permissions and Access Control

To protect your views from unauthorized access, Django uses Decorators for functions and Mixins for classes.

### 1. Function-Based: `@login_required`

This decorator automatically redirects the user to the login page if they aren't signed in.

```python
from django.contrib.auth.decorators import login_required

@login_required
def dashboard(request):
    return render(request, 'dashboard.html')

@login_required(login_url='/custom-login/')
def private_view(request):
    return render(request, 'private.html')
```

### 2. Class-Based: `LoginRequiredMixin`

In Class-Based Views, you inherit from a Mixin to enforce security.

```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import ListView

class PrivateListView(LoginRequiredMixin, ListView):
    model = Post
    template_name = 'private_posts.html'
    login_url = '/login/'
    redirect_field_name = 'next'
```

### 3. Permission-Based Access Control

```python
from django.contrib.auth.decorators import permission_required
from django.contrib.auth.mixins import PermissionRequiredMixin, UserPassesTestMixin

# Function-based view with permission
@permission_required('blog.add_post')
def create_post(request):
    return render(request, 'create_post.html')

# Class-based view with permission
class PostCreateView(PermissionRequiredMixin, CreateView):
    model = Post
    fields = ['title', 'content']
    permission_required = 'blog.add_post'

# Custom test mixin
class AuthorOnlyView(UserPassesTestMixin, DetailView):
    model = Post

    def test_func(self):
        post = self.get_object()
        return self.request.user == post.author
```

## Groups and Permissions

### Creating and Managing Groups

```python
from django.contrib.auth.models import Group, Permission
from django.contrib.contenttypes.models import ContentType
from .models import Post

# Create groups
editors_group, created = Group.objects.get_or_create(name='Editors')
authors_group, created = Group.objects.get_or_create(name='Authors')

# Get permissions
content_type = ContentType.objects.get_for_model(Post)
add_permission = Permission.objects.get(
    codename='add_post',
    content_type=content_type
)
change_permission = Permission.objects.get(
    codename='change_post',
    content_type=content_type
)

# Assign permissions to groups
editors_group.permissions.add(add_permission, change_permission)
authors_group.permissions.add(add_permission)

# Add user to group
user = User.objects.get(username='john_doe')
user.groups.add(editors_group)
```

### Checking Group Membership

```python
def check_group_membership(request):
    user = request.user

    if user.groups.filter(name='Editors').exists():
        # User is in Editors group
        return render(request, 'editor_dashboard.html')
    elif user.groups.filter(name='Authors').exists():
        # User is in Authors group
        return render(request, 'author_dashboard.html')
    else:
        return redirect('login')
```

## Optional: Advanced Authentication

### Custom Authentication Backend

```python
# authentication.py
from django.contrib.auth.backends import ModelBackend
from django.contrib.auth import get_user_model

User = get_user_model()

class EmailBackend(ModelBackend):
    def authenticate(self, request, username=None, password=None, **kwargs):
        email = kwargs.get('email') or username
        try:
            user = User.objects.get(email=email)
            if user.check_password(password) and user.is_active:
                return user
        except User.DoesNotExist:
            return None

    def get_user(self, user_id):
        try:
            return User.objects.get(pk=user_id)
        except User.DoesNotExist:
            return None

# settings.py
AUTHENTICATION_BACKENDS = [
    'myapp.authentication.EmailBackend',
    'django.contrib.auth.backends.ModelBackend',
]
```

### Two-Factor Authentication (2FA)

```python
# Install: pip install pyotp qrcode[pil]
import pyotp
import qrcode
from io import BytesIO
import base64

def setup_2fa(request):
    if request.method == 'POST':
        # Verify 2FA token
        token = request.POST.get('token')
        user_profile = request.user.userprofile

        if user_profile.verify_2fa_token(token):
            # Enable 2FA
            user_profile.two_factor_enabled = True
            user_profile.save()
            return redirect('2fa-success')
        else:
            return render(request, '2fa_setup.html', {'error': 'Invalid token'})
    else:
        # Generate 2FA secret
        user_profile = request.user.userprofile
        if not user_profile.two_factor_secret:
            user_profile.two_factor_secret = pyotp.random_base32()
            user_profile.save()

        # Generate QR code
        totp_uri = pyotp.totp.TOTP(user_profile.two_factor_secret).provisioning_uri(
            name=request.user.email,
            issuer_name='MyApp'
        )
        qr = qrcode.QRCode(version=1, box_size=10, border=5)
        qr.add_data(totp_uri)
        qr.make(fit=True)
        img = qr.make_image(fill_color="black", back_color="white")

        # Convert to base64 for template
        buffer = BytesIO()
        img.save(buffer, format='PNG')
        qr_code = base64.b64encode(buffer.getvalue()).decode()

        return render(request, '2fa_setup.html', {'qr_code': qr_code})
```

## Security Best Practices

Django is secure by default, but you must follow these rules for production:

### 1. Environment Configuration

```python
# settings.py
import os
from pathlib import Path

# Never commit SECRET_KEY to version control
SECRET_KEY = os.environ.get('SECRET_KEY', 'insecure-default-key')

# DEBUG mode
DEBUG = os.environ.get('DEBUG', 'False') == 'True'

# Allowed hosts
ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

# Database credentials from environment
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}
```

### 2. Security Settings

```python
# Security headers
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'

# HTTPS settings (for production)
SECURE_SSL_REDIRECT = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

# Session security
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SESSION_COOKIE_HTTPONLY = True
CSRF_COOKIE_HTTPONLY = True
```

### 3. Password Security

```python
# Password validation
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        'OPTIONS': {
            'min_length': 12,
        }
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]

# Password hashing
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
]
```

### 4. CSRF Protection

Always include `{% csrf_token %}` in your POST forms:

```html
<form method="post">
    {% csrf_token %}
    <!-- form fields -->
</form>
```

### 5. SQL Injection Protection

The Django ORM automatically protects you from SQL Injection:

```python
# Safe - uses parameterized queries
Post.objects.filter(title=title, author=user)

# Dangerous - never do this
cursor.execute(f"SELECT * FROM blog_post WHERE title = '{title}'")
```

### 6. XSS Protection

Django templates automatically escape HTML:

```html
<!-- Safe - HTML is escaped -->
{{ user_input }}

<!-- Unsafe - only use with trusted content -->
{{ trusted_content|safe }}
```

## Security Middleware

Django includes several security middleware classes:

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

## Security Testing

### Testing Authentication

```python
from django.test import TestCase, Client
from django.contrib.auth import get_user_model

User = get_user_model()

class AuthTests(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        self.client = Client()

    def test_login(self):
        response = self.client.post('/login/', {
            'username': 'testuser',
            'password': 'testpass123'
        })
        self.assertEqual(response.status_code, 302)

    def test_protected_view(self):
        self.client.login(username='testuser', password='testpass123')
        response = self.client.get('/dashboard/')
        self.assertEqual(response.status_code, 200)

    def test_unauthorized_access(self):
        response = self.client.get('/dashboard/')
        self.assertEqual(response.status_code, 302)  # Redirect to login
```

## Best Practices

1. **Always use HTTPS in production**.
   HTTPS protects credentials and session cookies in transit. Without it, even a strong password can be intercepted on an insecure network.
2. **Never commit secrets to version control**.
   Keys and credentials inevitably leak if stored in Git. Treat them like passwords and load them from environment variables or a secrets manager.
3. **Use environment variables for sensitive configuration**.
   This keeps the same codebase deployable across environments (dev/staging/prod) without hardcoding values into `settings.py`.
4. **Enable security middleware and headers**.
   Django’s security middleware sets headers that mitigate common browser-based attacks. It’s one of the highest ROI security features you can turn on.
5. **Validate all user input on the server**.
   Client-side checks are for UX. Security requires server-side validation and safe database access patterns (which Django’s ORM provides).
6. **Use Django’s built-in authentication instead of rolling your own**.
   The built-in system is widely audited and integrates with sessions, middleware, permissions, and admin. Custom login systems often introduce subtle vulnerabilities.
7. **Log security-relevant events**.
   Login failures, permission-denied events, and suspicious activity should be visible in logs so you can detect abuse early.
8. **Keep Django updated**.
   Security patches land regularly. Staying current reduces exposure time for known vulnerabilities.

## Summary

- **User Model**: A pre-built class for handling usernames, passwords, and emails
- **Auth Views**: Built-in logic for login/logout and password resets
- **Decorators/Mixins**: Tools to restrict access to specific pages based on login status
- **Security**: Django handles CSRF, XSS, and SQL Injection automatically
- **Groups & Permissions**: Fine-grained access control system
- **2FA**: Additional security layer for sensitive applications

## Important Keywords

### **Authentication**

Process of verifying user identity, typically through username/password or other credentials.

### **Authorization**

Process of determining if an authenticated user has permission to perform specific actions.

### **User Model**

Django's built-in model for handling user accounts with authentication and authorization features.

### **Custom User Model**

User-defined user model that extends Django's base User model for additional fields and functionality.

### **LoginRequiredMixin**

Mixin for Class-Based Views that requires users to be authenticated to access the view.

### **@login_required**

Decorator for Function-Based Views that restricts access to authenticated users only.

### **Permission**

Fine-grained access control that determines what actions a user can perform on specific objects.

### **Group**

Collection of users with shared permissions, simplifying permission management for multiple users.

### **CSRF Token**

Security feature that prevents cross-site request forgery attacks by validating form submissions.

### **Session**

Mechanism for storing information about a specific user across multiple requests.

### **Middleware**

Hooks that process requests and responses globally, including security middleware for protection.

### **Password Hashing**

Process of converting passwords into secure hash values for storage, never storing plain text passwords.

### **Two-Factor Authentication (2FA)**

Security process requiring two different forms of identification to verify user identity.

### **Security Headers**

HTTP headers that enhance security by controlling browser behavior and preventing attacks.

### **SQL Injection**

Security vulnerability where malicious SQL code is inserted into queries, prevented by Django's ORM.

### **Cross-Site Scripting (XSS)**

Security vulnerability where malicious scripts are injected into web pages, prevented by Django's template auto-escaping.

### **Authentication Backend**

System that authenticates users against different sources (database, LDAP, etc.).

### **Password Validator**

Component that enforces password complexity and security requirements.

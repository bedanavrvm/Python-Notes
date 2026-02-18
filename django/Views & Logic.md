# 3. Views & Logic

In the MVT pattern, the View is the bridge between the Model and the Template. It receives an HttpRequest, performs the necessary logic (like fetching data or processing a form), and returns an HttpResponse.

## Function-Based Views (FBV)

Function-Based Views are the simplest way to write views. They are just Python functions that take a request object as their first argument.

### The Basic Structure

Every view must return an HttpResponse object. Django provides a `render()` shortcut to combine a template with a data dictionary (the context).

```python
from django.shortcuts import render
from .models import Post

def post_list(request):
    # Logic: Fetch data from the Model
    posts = Post.objects.all()

    # Logic: Prepare data for the Template
    context = {'posts': posts}

    # Return: Render the template with the context
    return render(request, 'blog/post_list.html', context)
```

### Handling Methods (GET vs POST)

In an FBV, you use a simple if statement to handle different HTTP methods.

```python
from django.http import HttpResponse

def contact_view(request):
    if request.method == "POST":
        # Process form data
        name = request.POST.get('name')
        return HttpResponse(f"Thanks {name}!")

    # Otherwise, it's a GET; just show the page
    return render(request, 'contact.html')
```

### URL Parameters

Views can receive parameters from the URL routing system:

```python
def post_detail(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    return render(request, 'blog/detail.html', {'post': post})
```

## Class-Based Views (CBV)

As your project grows, you'll find yourself repeating the same logic (fetching a list, showing a detail page). Class-Based Views allow you to use Inheritance to reduce boilerplate code.

### 1. The Generic Views

Django provides "Generic" classes for common tasks:

* **ListView**: To display a list of objects.
* **DetailView**: To display a single object.
* **CreateView / UpdateView**: To handle forms and database saving.
* **DeleteView**: To handle object deletion.
* **TemplateView**: To render a template without models.

```python
from django.views.generic import ListView, DetailView
from .models import Post

class PostListView(ListView):
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts'  # Default is 'object_list'
    paginate_by = 10  # Add pagination

    def get_queryset(self):
        # Custom filtering
        return Post.objects.filter(is_published=True)

class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/post_detail.html'
    slug_url_kwarg = 'post_slug'  # Use slug instead of pk
    query_pk_and_slug = True
```

### 2. Custom Class-Based Views

Create your own CBVs for reusable logic:

```python
from django.views.generic import View
from django.contrib.auth.mixins import LoginRequiredMixin

class AuthorPostsView(LoginRequiredMixin, View):
    def get(self, request, author_id):
        posts = Post.objects.filter(author_id=author_id)
        return render(request, 'blog/author_posts.html', {'posts': posts})

    def post(self, request, author_id):
        # Handle POST requests
        action = request.POST.get('action')
        if action == 'delete_all':
            Post.objects.filter(author_id=author_id).delete()
        return redirect('author-posts', author_id=author_id)
```

### 3. CBV Mixins

Mixins add reusable functionality to CBVs:

```python
from django.contrib.auth.mixins import LoginRequiredMixin, UserPassesTestMixin
from django.views.generic.edit import CreateView, UpdateView

class PostCreateView(LoginRequiredMixin, CreateView):
    model = Post
    fields = ['title', 'content']
    template_name = 'blog/post_form.html'

    def form_valid(self, form):
        form.instance.author = self.request.user
        return super().form_valid(form)

class PostUpdateView(UserPassesTestMixin, UpdateView):
    model = Post
    fields = ['title', 'content']

    def test_func(self):
        post = self.get_object()
        return self.request.user == post.author
```

### 4. When to use CBV vs FBV?

* **Use FBVs** for complex, custom logic that doesn't fit a standard pattern. They are easier to read for beginners.
* **Use CBVs** for standard CRUD (Create, Read, Update, Delete) operations. They make your code much shorter and more modular.

## The Request and Response Objects

Understanding what lives inside these objects is key to mastering views.

### The HttpRequest Object

When a view is called, Django passes an instance containing metadata about the request:

```python
def debug_request(request):
    # User information
    user = request.user  # Currently logged-in user or AnonymousUser

    # Request metadata
    method = request.method  # 'GET', 'POST', 'PUT', etc.
    path = request.path      # URL path without domain
    full_path = request.get_full_path()  # Including query parameters

    # Data access
    get_data = request.GET      # Query parameters (from URL)
    post_data = request.POST    # Form data (from POST body)
    files = request.FILES       # Uploaded files
    cookies = request.COOKIES   # Cookies
    headers = request.headers   # HTTP headers

    # Security
    is_secure = request.is_secure  # HTTPS?
    is_ajax = request.headers.get('x-requested-with') == 'XMLHttpRequest'

    return HttpResponse(f"Method: {method}, Path: {path}")
```

### The HttpResponse Object

You don't always have to render HTML. You can return other types of data:

```python
from django.http import JsonResponse, HttpResponseRedirect, HttpResponseNotFound
from django.shortcuts import redirect

def api_view(request):
    data = {'message': 'Hello API!', 'status': 'success'}
    return JsonResponse(data)

def redirect_view(request):
    return redirect('home-page')  # Uses reverse() internally

def custom_response(request):
    response = HttpResponse("Custom content")
    response['X-Custom-Header'] = 'Value'
    response.set_cookie('test', 'value')
    return response

def error_view(request):
    return HttpResponseNotFound("Page not found")
```

## Form Handling

Django provides powerful form handling capabilities.

### Using Django Forms\`\`\`python

from django import forms from .models import Post

class PostForm(forms.ModelForm): class Meta: model = Post fields = \['title', 'content', 'is\_published'] widgets = { 'content': forms.Textarea(attrs={'rows': 10}), } `</div><div data-gb-custom-block data-tag="tab" data-title='views.py'>`python from django.shortcuts import redirect, render

from .forms import PostForm

def create\_post(request): if request.method == 'POST': form = PostForm(request.POST) if form.is\_valid(): post = form.save(commit=False) post.author = request.user post.save() return redirect('post-detail', pk=post.pk) else: form = PostForm()

```
return render(request, 'blog/create_post.html', {'form': form})
```

````</div></div>###

```python
from django.views.generic.edit import CreateView, UpdateView

class PostCreateView(CreateView):
    model = Post
    form_class = PostForm
    template_name = 'blog/post_form.html'
    success_url = '/blog/posts/'  # Or use reverse_lazy

    def form_valid(self, form):
        form.instance.author = self.request.user
        return super().form_valid(form)
````

## Security in Views

### CSRF Protection

Django protects against Cross-Site Request Forgery attacks:

````html
<!-- In templates -->
<form method="post">
    {% csrf_token %}
    <!-- form fields -->
</form>
```### Authentication and Permissions<div data-gb-custom-block data-tag="tabs"><div data-gb-custom-block data-tag="tab" data-title='FBV (decorators)'>```python
from django.contrib.auth.decorators import login_required, permission_required
from django.shortcuts import render

@login_required
def my_protected_view(request):
    return render(request, 'protected.html')

@permission_required('blog.add_post')
def create_post_view(request):
    return render(request, 'create_post.html')
```</div><div data-gb-custom-block data-tag="tab" data-title='CBV (mixins)'>```python
from django.contrib.auth.mixins import LoginRequiredMixin, UserPassesTestMixin
from django.shortcuts import render
from django.views import View

class ProtectedView(LoginRequiredMixin, View):
    login_url = '/login/'

    def get(self, request):
        return render(request, 'protected.html')

class AdminOnlyView(UserPassesTestMixin, View):
    def test_func(self):
        return self.request.user.is_superuser

    def get(self, request):
        return render(request, 'admin_only.html')
```</div></div>### Input Validation

```python
def safe_view(request):
    # Never trust user input
    user_input = request.GET.get('search', '')

    # Use Django's validation
    from django.core.validators import validate_email
    from django.core.exceptions import ValidationError

    try:
        validate_email(user_input)
        # Valid email
    except ValidationError:
        # Invalid email
        return HttpResponse("Invalid email", status=400)
````

## Error Handling

### Custom Error Views

```python
# urls.py
handler404 = 'myapp.views.custom_404'
handler500 = 'myapp.views.custom_500'

# views.py
def custom_404(request, exception):
    return render(request, 'errors/404.html', status=404)

def custom_500(request):
    return render(request, 'errors/500.html', status=500)
```

### Exception Handling in Views

```python
import logging
from django.db import DatabaseError

logger = logging.getLogger(__name__)

def risky_operation(request):
    try:
        # Potentially failing operation
        result = some_risky_function()
        return render(request, 'success.html', {'result': result})
    except DatabaseError:
        logger.error("Database error occurred")
        return render(request, 'error.html', {'error': 'Database error'})
    except Exception as e:
        logger.error(f"Unexpected error: {e}")
        return render(request, 'error.html', {'error': 'Something went wrong'})
```

## Optional: API Development with Django

### Building API Endpoints

```python
from django.http import JsonResponse
from django.views.decorators.http import require_http_methods
from django.core import serializers

@require_http_methods(["GET", "POST"])
def posts_api(request):
    if request.method == 'GET':
        posts = Post.objects.all()
        data = [
            {'id': p.id, 'title': p.title, 'content': p.content[:100]}
            for p in posts
        ]
        return JsonResponse({'posts': data})

    elif request.method == 'POST':
        title = request.POST.get('title')
        content = request.POST.get('content')

        if title and content:
            post = Post.objects.create(
                title=title,
                content=content,
                author=request.user
            )
            return JsonResponse({'id': post.id, 'status': 'created'}, status=201)

        return JsonResponse({'error': 'Missing data'}, status=400)
```

### Status Codes and Headers

```python
from django.http import HttpResponse

def api_response_examples(request):
    # Different status codes
    if request.GET.get('type') == 'created':
        return HttpResponse('Created', status=201)
    elif request.GET.get('type') == 'bad_request':
        return HttpResponse('Bad Request', status=400)
    elif request.GET.get('type') == 'unauthorized':
        return HttpResponse('Unauthorized', status=401)

    # Custom headers
    response = HttpResponse('API Response')
    response['Content-Type'] = 'application/json'
    response['X-API-Version'] = '1.0'
    return response
```

## Optional: Performance Optimization

### Query Optimization

```python
def optimized_view(request):
    # Avoid N+1 queries with select_related/prefetch_related
    posts = Post.objects.select_related('author').prefetch_related('tags')

    # Only fetch needed fields
    posts = Post.objects.only('id', 'title', 'published_date')

    return render(request, 'posts.html', {'posts': posts})
```

### Caching

```python
from django.views.decorators.cache import cache_page
from django.core.cache import cache

@cache_page(60 * 15)  # Cache for 15 minutes
def expensive_view(request):
    # Expensive operation
    result = complex_calculation()
    return render(request, 'result.html', {'result': result})

def manual_cache_view(request):
    cache_key = f'view_data_{request.user.id}'
    data = cache.get(cache_key)

    if data is None:
        data = expensive_operation()
        cache.set(cache_key, data, 60 * 30)  # 30 minutes

    return render(request, 'data.html', {'data': data})
```

## Shortcuts: render and get\_object\_or\_404

Django provides shortcuts to make views more concise and robust.

### get\_object\_or\_404

Instead of writing a try/except block to catch objects that don't exist, use this:

```python
from django.shortcuts import get_object_or_404

def post_detail(request, pk):
    # Automatically returns a 404 page if the post doesn't exist
    post = get_object_or_404(Post, pk=pk)
    return render(request, 'blog/detail.html', {'post': post})
```

### Other Useful Shortcuts

```python
from django.shortcuts import render, redirect, get_list_or_404

def post_list(request):
    # Get list or return 404 if empty
    posts = get_list_or_404(Post, is_published=True)
    return render(request, 'blog/list.html', {'posts': posts})

def create_and_redirect(request):
    # Create object and redirect
    post = Post.objects.create(
        title="New Post",
        content="Content here",
        author=request.user
    )
    return redirect(post)  # Uses get_absolute_url if defined
```

## Mini Walkthrough: Post List → Post Detail (URL → View → Template)

This walkthrough connects routing and views, and shows how context flows into templates.

<details>

<summary>Show walkthrough code</summary>

#### 1. URLs: define list + detail routes

```python
# blog/urls.py
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    path('posts/', views.post_list, name='list'),
    path('posts/<int:pk>/', views.post_detail, name='detail'),
]
```

#### 2. Views: fetch data and render templates

```python
# blog/views.py
from django.shortcuts import get_object_or_404, render
from .models import Post

def post_list(request):
    posts = Post.objects.all().order_by('-published_date')
    return render(request, 'blog/post_list.html', {'posts': posts})

def post_detail(request, pk):
    post = get_object_or_404(Post, pk=pk)
    return render(request, 'blog/post_detail.html', {'post': post})
```

#### 3. Template: generate links using URL names (no hardcoding)

```html
<!-- blog/templates/blog/post_list.html -->
{% for post in posts %}
  <a href="{% url 'blog:detail' pk=post.pk %}">{{ post.title }}</a>
{% endfor %}
```

</details>

## Best Practices

1. **Keep views thin**. A view should coordinate work, not contain all the business logic. As logic grows, push it into model methods, services, or utility functions so views stay readable and testable.
2. **Use CBVs for standard patterns and FBVs for custom logic**. CBVs (especially generic ones) reduce boilerplate for CRUD, pagination, and forms. FBVs are often clearer when you have a one-off flow that doesn’t fit a reusable pattern.
3. **Always validate user input**. Never trust `request.GET`/`request.POST` directly. Prefer Django Forms and model validation, and always handle invalid data explicitly.
4. **Handle exceptions gracefully and log errors**. Users should see helpful error pages; developers should get logs with enough context to debug. Avoid leaking stack traces or sensitive data in production.
5. **Optimize database queries**. Watch out for the N+1 query pattern when templates loop over related objects. Use `select_related()` and `prefetch_related()` when you know you’ll need related data.
6. **Use appropriate HTTP status codes**. Status codes are part of your API contract and make debugging easier. For example, return 201 on create, 400 on validation errors, and 404 when a resource doesn’t exist.
7. **Implement authentication and authorization intentionally**. Authentication answers “who are you?”; authorization answers “are you allowed to do this?”. Use decorators/mixins and Django permissions consistently.
8. **Cache expensive operations**. Cache is a tool to reduce load, but it adds complexity. Cache only after you understand what’s slow, and choose a cache key strategy that avoids serving the wrong user’s data.

## Summary

* Views handle request processing and response generation
* FBVs are simple and explicit, CBVs are reusable and organized
* Django provides shortcuts to reduce boilerplate code
* Security features protect against common attacks
* Proper error handling ensures robust applications

## Important Keywords

### **Function-Based View (FBV)**

Simple Python function that handles a web request and returns a response, offering explicit control over logic flow.

### **Class-Based View (CBV)**

Python class that handles web requests using methods for different HTTP verbs, providing reusable and organized code structure.

### **Generic View**

Pre-built CBV classes in Django that implement common patterns like list, detail, create, update, and delete operations.

### **Mixin**

Reusable class that provides specific functionality to CBVs when combined through multiple inheritance.

### **HttpRequest**

Object containing metadata about the incoming request, including method, headers, user data, and request parameters.

### **HttpResponse**

Object representing the response sent back to the client, containing content, status code, and headers.

### **JsonResponse**

Specialized HttpResponse that returns JSON-encoded data, commonly used for API endpoints.

### **render()**

Django shortcut that combines a template with context data and returns an HttpResponse.

### **get\_object\_or\_404()**

Shortcut that retrieves an object or raises Http404 exception if not found, simplifying error handling.

### **CSRF Token**

Security feature that prevents cross-site request forgery attacks by validating form submissions.

### **Authentication**

Process of verifying user identity, typically through login decorators or mixins.

### **Authorization**

Process of determining if an authenticated user has permission to perform specific actions.

### **Form Validation**

Process of ensuring submitted data meets specified requirements before processing.

### **Status Code**

HTTP status code indicating the result of a request (200 OK, 404 Not Found, 201 Created, etc.).

### **QuerySet Optimization**

Techniques like select\_related() and prefetch\_related() to reduce database queries and improve performance.

### **Caching**

Storing computed results or database queries temporarily to improve response times.

### **Middleware**

Hooks that process requests and responses globally before they reach views or after leaving views.

### **Decorator**

Function that modifies or enhances another function, commonly used for authentication and permission checks.


# 2. Routing (The URL Dispatcher)

The URL Dispatcher is the first point of contact when a request enters a Django application. It acts as a traffic controller, mapping a requested URL (like `/blog/post/5/`) to a specific logic handler (the View).

```mermaid
flowchart LR
    A[Incoming URL] --> B[Project urls.py]
    B -->|include()| C[App urls.py]
    C -->|path() match| D[View function / CBV]
    D --> E[HttpResponse / redirect]
```

## URL Patterns

In Django, URLs are defined in `urls.py` files using the `path()` function. This function links a string pattern to a view function.

### The path() Function

The `path()` function takes four arguments (two required, two optional):

- **route**: The string pattern (the URL).
- **view**: The function or class-based view to execute.
- **kwargs**: (Optional) Extra arguments passed to the view.
- **name**: (Optional) A unique name for the URL, used for "reverse" lookups.

```python
from django.urls import path
from . import views

urlpatterns = [
    path('about/', views.about_page, name='about'),
]
```

## Dynamic Routing & Path Converters

Hardcoding every single URL (like `/post/1/`, `/post/2/`) is impossible for large sites. Django uses Path Converters to capture values from the URL and pass them as arguments to the view.

### Syntax

Use angle brackets `<type:variable_name>` to define a dynamic segment.

| Converter | Description |
|-----------|-------------|
| `str` | Matches any non-empty string, excluding path separator `/`. (Default) |
| `int` | Matches zero or any positive integer. |
| `slug` | Matches any slug string (ASCII letters/numbers, plus hyphens/underscores). |
| `uuid` | Matches a formatted UUID. |
| `path` | Matches any non-empty string, including path separator `/`. |

{% tabs %}

{% tab title="urls.py" %}

```python
# urls.py
path('post/<int:post_id>/', views.post_detail, name='post-detail'),
```

{% endtab %}

{% tab title="views.py" %}

```python
# views.py
def post_detail(request, post_id):
    # 'post_id' is automatically passed as an integer argument
    return HttpResponse(f"Displaying post number {post_id}")
```

{% endtab %}

{% endtabs %}

## App-Level vs. Project-Level URLs

To keep the "pluggable" philosophy, we don't put every URL in the main project folder. Instead, each App manages its own `urls.py`, and the main project "includes" them.

### Project-Level (`my_project/urls.py`)

{% tabs %}

{% tab title="project urls.py" %}

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls')),  # Delegation to app
]
```

{% endtab %}

{% tab title="app urls.py" %}

```python
from django.urls import path
from . import views

urlpatterns = [
    path('latest/', views.recent_posts),
]
```

{% endtab %}

{% endtabs %}

Resulting URL: `www.example.com/blog/latest/`

## Namespacing and Reversing

You should never hardcode URLs in your templates (e.g., `<a href="/blog/post/5/">`). If you change your URL structure later, your links will break. Instead, you "Reverse" the URL using its name.

### Why use `app_name`?

If you have two apps (blog and store) that both have a view named `index`, Django won't know which one to pick. Namespacing solves this.

```python
# blog/urls.py
app_name = 'blog'
urlpatterns = [
    path('archive/', views.archive, name='post-archive'),
]
```

### How to Reverse

#### In Templates: Use the `{% url %}` tag

{% tabs %}

{% tab title="Template" %}

```html
<a href="{% url 'blog:post-archive' %}">Go to Archive</a>
```

{% endtab %}

{% tab title="View (Python)" %}

```python
from django.shortcuts import redirect
from django.urls import reverse

def my_view(request):
    return redirect(reverse('blog:post-archive'))
```

{% endtab %}

{% endtabs %}

## Advanced Routing Patterns

### Including URL Conf with Extra Namespace

You can include apps with an additional namespace for even better organization:

```python
# project/urls.py
urlpatterns = [
    path('api/v1/', include(('api.urls', 'api'), namespace='api')),
]
```

### Reversing with Parameters

Pass dynamic data when reversing URLs:

{% tabs %}

{% tab title="View (Python)" %}

```python
# In views
def create_post(request, post_id):
    return redirect(reverse('blog:post-detail', kwargs={'post_id': post_id}))
```

{% endtab %}

{% tab title="Template" %}

```html
<!-- In templates -->
<a href="{% url 'blog:post-detail' post_id=post.id %}">Read More</a>
```

{% endtab %}

{% endtabs %}

### Custom Path Converters

Create your own converters for specialized URL patterns:

{% tabs %}

{% tab title="converters.py" %}

```python
# converters.py
class YearConverter:
    regex = r'\d{4}'

    def to_python(self, value):
        return int(value)

    def to_url(self, value):
        return f'{value:04d}'
```

{% endtab %}

{% tab title="urls.py" %}

```python
# urls.py
from django.urls import path, register_converter

from . import views
from .converters import YearConverter

register_converter(YearConverter, 'year')

urlpatterns = [
    path('archive/<year:year>/', views.year_archive),
]
```

{% endtab %}

{% endtabs %}

## Mini Walkthrough: Blog Detail Page (URL → View → Template Link)

This mini walkthrough ties routing to views and templates, and shows why naming URLs matters.

<details>
<summary>Show walkthrough code</summary>

### 1. Define the URL pattern (and name it)

```python
# blog/urls.py
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    path('posts/<slug:slug>/', views.post_detail, name='detail'),
]
```

### 2. Use the URL parameter in your view

```python
# blog/views.py
from django.shortcuts import get_object_or_404, render
from .models import Post

def post_detail(request, slug):
    post = get_object_or_404(Post, slug=slug)
    return render(request, 'blog/post_detail.html', {'post': post})
```

### 3. Generate links safely from templates

```html
<!-- blog/templates/blog/post_list.html -->
{% for post in posts %}
  <a href="{% url 'blog:detail' post.slug %}">{{ post.title }}</a>
{% endfor %}
```

</details>

If you later change the URL from `posts/<slug:slug>/` to something else, **every template link keeps working** as long as the URL name (`blog:detail`) stays the same.

## Best Practices

1. **Always name your URLs** for reversability.
   A URL name is the stable identifier you reference from templates (`{% url %}`) and Python (`reverse()`). If you hardcode paths, any future refactor breaks links site-wide.
2. **Use `app_name`** in app URLconfs to prevent conflicts.
   Namespacing becomes essential once you have multiple apps. It prevents collisions like two different apps both defining `name='detail'`.
3. **Keep URLs RESTful** for APIs (GET/POST/PUT/DELETE patterns).
   Consistent URL shapes make APIs easier to understand, document, and consume. For example, `/api/posts/` for list/create and `/api/posts/<id>/` for retrieve/update/delete.
4. **Use specific converters** (`int`, `slug`) for validation.
   Converters are lightweight validation at the routing layer. If an ID must be numeric, using `<int:id>` prevents accidental matches and gives cleaner 404 behavior.
5. **Group related URLs** under common prefixes.
   This keeps the URL map readable and helps avoid deep, scattered URL patterns as the project grows (e.g., group everything blog-related under `blog/`).
6. **Avoid trailing slash inconsistencies**.
   Pick a convention and stick to it. Django often uses trailing slashes by default, and `APPEND_SLASH = True` can help normalize requests, but relying on it to “fix” inconsistent patterns can hide routing mistakes.

## Summary

- `path()` connects URLs to Views.
- Path Converters like `<int:pk>` allow for dynamic, data-driven URLs.
- `include()` keeps your project organized by delegating routing to individual apps.
- Namespacing and Naming URLs allow you to change your URL structure without breaking links across your site.

## Important Keywords

### **URL Dispatcher**

Django’s mechanism for mapping incoming URLs to appropriate view functions or classes using URL patterns defined in `urls.py`.

### **path()**

Function that defines a URL pattern and maps it to a view, with optional parameters for kwargs and naming.

### **Path Converter**

Special syntax `<type:variable>` that captures URL segments and converts them to specified Python types.

### **Dynamic Routing**

Technique using path converters to capture variable values from URLs and pass them as arguments to views.

### **URLconf (`urls.py`)**

File containing URL pattern definitions that Django uses to route requests to the appropriate view.

### **include()**

Function that includes another URLconf module, allowing apps to manage their own URL patterns independently.

### **Namespacing**

Using `app_name` and URL names to prevent conflicts between different apps with similar view names.

### **Reversing URLs**

Generating URLs from their names instead of hardcoding them, allowing URL structure changes without breaking links.

### **reverse()**

Python function that returns the URL for a given view name and optional parameters.

### **{% url %}**

Django template tag used to reverse URLs within templates, supporting namespacing and parameters.

### **Slug**

URL-friendly string converter that matches lowercase letters, numbers, hyphens, and underscores, commonly used for SEO-friendly URLs.

### **UUID Converter**

Path converter that matches universally unique identifiers in the standard UUID format.

### **Path Converter (`path`)**

Special converter that matches any non-empty string, including path separators, for capturing full paths.

### **Custom Converter**

User-defined path converter for handling specialized URL patterns with custom validation and conversion logic.

### **RESTful Routing**

Designing URL patterns that follow REST principles using HTTP methods (GET, POST, PUT, DELETE) for API endpoints.

### **APPEND_SLASH**

Django setting that determines whether URLs should end with a trailing slash, ensuring consistency across the site.

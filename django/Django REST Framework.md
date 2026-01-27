
# 8. Django REST Framework (DRF)

As modern web development shifts toward decoupled architectures (where a frontend like React or Vue talks to a backend via JSON), the Django REST Framework (DRF) has become the industry standard for building powerful and flexible Web APIs.

## What is a REST API?

A REST (Representational State Transfer) API allows different systems to communicate over HTTP using standard methods. Instead of returning HTML templates, the server returns JSON data.

- **GET**: Retrieve data
- **POST**: Create data
- **PUT/PATCH**: Update data
- **DELETE**: Remove data

## Installation and Setup

```bash
# Install DRF
pip install djangorestframework

# Add to settings.py
INSTALLED_APPS = [
    # ... other apps
    'rest_framework',
]
```

## Serializers: The Heart of DRF

In standard Django, the View sends a Python Object to a Template. In DRF, we need to convert that Python Object into JSON. This process is called Serialization.

### Basic ModelSerializer

```python
# serializers.py
from rest_framework import serializers
from .models import Post

class PostSerializer(serializers.ModelSerializer):
    author_name = serializers.CharField(source='author.username', read_only=True)

    class Meta:
        model = Post
        fields = ['id', 'title', 'content', 'author_name', 'created_at']
        read_only_fields = ['created_at']
```

### Custom Serializer Methods

```python
class PostSerializer(serializers.ModelSerializer):
    author_name = serializers.CharField(source='author.username', read_only=True)
    word_count = serializers.SerializerMethodField()

    class Meta:
        model = Post
        fields = ['id', 'title', 'content', 'author_name', 'word_count']

    def get_word_count(self, obj):
        return len(obj.content.split())

    def validate_title(self, value):
        if len(value) < 5:
            raise serializers.ValidationError("Title must be at least 5 characters")
        return value
```

### Nested Serializers

```python
class AuthorSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email']

class PostSerializer(serializers.ModelSerializer):
    author = AuthorSerializer(read_only=True)
    comments_count = serializers.SerializerMethodField()

    class Meta:
        model = Post
        fields = ['id', 'title', 'content', 'author', 'comments_count']

    def get_comments_count(self, obj):
        return obj.comments.count()
```

## API Views

DRF provides several ways to build views, ranging from manual control to total automation.

### 1. APIView (Manual Control)

Similar to Django's View class but tailored for REST.

```python
# views.py
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from .models import Post
from .serializers import PostSerializer

class PostListAPI(APIView):
    def get(self, request):
        posts = Post.objects.all()
        serializer = PostSerializer(posts, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = PostSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

class PostDetailAPI(APIView):
    def get_object(self, pk):
        try:
            return Post.objects.get(pk=pk)
        except Post.DoesNotExist:
            raise Http404

    def get(self, request, pk):
        post = self.get_object(pk)
        serializer = PostSerializer(post)
        return Response(serializer.data)

    def put(self, request, pk):
        post = self.get_object(pk)
        serializer = PostSerializer(post, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk):
        post = self.get_object(pk)
        post.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### 2. Generic Views (Semi-Automated)

DRF provides generic views that handle common patterns.

```python
from rest_framework import generics
from .models import Post
from .serializers import PostSerializer

class PostListCreateView(generics.ListCreateAPIView):
    queryset = Post.objects.all()
    serializer_class = PostSerializer

    def perform_create(self, serializer):
        serializer.save(author=self.request.user)

class PostRetrieveUpdateDestroyView(generics.RetrieveUpdateDestroyAPIView):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```

### 3. ViewSets (Fully Automated)

If you want to perform standard CRUD operations, ViewSets allow you to handle all actions (list, create, retrieve, update, delete) in a single class.

```python
from rest_framework import viewsets
from rest_framework.permissions import IsAuthenticatedOrReadOnly
from .models import Post
from .serializers import PostSerializer

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]

    def perform_create(self, serializer):
        serializer.save(author=self.request.user)

    def get_queryset(self):
        queryset = Post.objects.all()
        author = self.request.query_params.get('author', None)
        if author is not None:
            queryset = queryset.filter(author__id=author)
        return queryset
```

## Routers

When using ViewSets, you don't need to manually define paths for every action. DRF's Routers automatically generate the URL configuration for you.

```python
# urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import PostViewSet

router = DefaultRouter()
router.register(r'posts', PostViewSet)

urlpatterns = [
    path('api/', include(router.urls)),
]
```

This automatically creates URLs like:

- `/api/posts/` (GET/POST) - List and create posts
- `/api/posts/{id}/` (GET/PUT/DELETE) - Retrieve, update, delete post

## Mini Walkthrough: Build a Simple Posts API (Serializer → ViewSet → Router)

This walkthrough shows the smallest “real” DRF setup you’ll use in most projects.

{% tabs %}

{% tab title="serializers.py" %}

```python
# api/serializers.py
from rest_framework import serializers
from blog.models import Post

class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = ['id', 'title', 'slug', 'content']
```

{% endtab %}

{% tab title="views.py" %}

```python
# api/views.py
from rest_framework import viewsets
from blog.models import Post
from .serializers import PostSerializer

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```

{% endtab %}

{% tab title="urls.py" %}

```python
# api/urls.py
from django.urls import include, path
from rest_framework.routers import DefaultRouter
from .views import PostViewSet

router = DefaultRouter()
router.register(r'posts', PostViewSet, basename='post')

urlpatterns = [
    path('', include(router.urls)),
]
```

{% endtab %}

{% tab title="test" %}
If your project includes `api/urls.py` under `/api/`, you can test:

`GET /api/posts/` to list posts

`POST /api/posts/` with JSON to create one
{% endtab %}

{% endtabs %}

### Custom Actions in ViewSets

<details>
<summary>Show custom actions example</summary>

```python
from rest_framework.decorators import action
from rest_framework.response import Response

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer

    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        post = self.get_object()
        post.is_published = True
        post.save()
        return Response({'status': 'post published'})

    @action(detail=False, methods=['get'])
    def published(self, request):
        published_posts = self.queryset.filter(is_published=True)
        serializer = self.get_serializer(published_posts, many=True)
        return Response(serializer.data)
```

</details>

## Authentication and Permissions in DRF

DRF allows you to set security policies at the global or view level.

### Global Authentication Settings

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
}
```

### Token Authentication

```python
# Install: pip install djangorestframework-authtoken
# Add to settings.py
INSTALLED_APPS = [
    # ...
    'rest_framework.authtoken',
]

# Create tokens for users
from rest_framework.authtoken.models import Token
from django.contrib.auth.models import User

for user in User.objects.all():
    Token.objects.get_or_create(user=user)
```

### View-Level Permissions

```python
from rest_framework import permissions

class PostViewSet(viewsets.ModelViewSet):
    # Only logged-in users can edit; others can only view
    permission_classes = [IsAuthorOrReadOnly]

class IsAuthorOrReadOnly(permissions.BasePermission):
    def has_object_permission(self, request, view, obj):
        if request.method in permissions.SAFE_METHODS:
            return True
        return obj.author == request.user
```

### Permission Classes

```python
# Built-in permission classes
permissions.IsAuthenticated          # Must be logged in
permissions.IsAdminUser              # Must be staff/superuser
permissions.IsAuthenticatedOrReadOnly  # Read-only for unauthenticated
permissions.AllowAny                 # No restrictions
permissions.DjangoModelPermissions   # Django permissions
permissions.DjangoObjectPermissions  # Per-object permissions
```

## Request and Response Objects

### Request Object

```python
def api_view(request):
    # Access request data
    data = request.data  # POST/PUT/PATCH data
    query_params = request.query_params  # GET parameters
    user = request.user  # Authenticated user

    # Content negotiation
    if request.accepted_renderer.format == 'json':
        return Response({'message': 'JSON response'})

    return Response({'message': 'Default response'})
```

### Response Object

```python
from rest_framework.response import Response
from rest_framework import status

def api_response(request):
    # Different response types
    return Response({'data': 'success'})  # 200 OK
    return Response({'error': 'Not found'}, status=status.HTTP_404_NOT_FOUND)
    return Response({'created': True}, status=status.HTTP_201_CREATED)
    return Response(status=status.HTTP_204_NO_CONTENT)
```

## Optional: Pagination

DRF provides built-in pagination for list views.

### Global Pagination Settings

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
}
```

### Custom Pagination

```python
from rest_framework.pagination import PageNumberPagination

class StandardResultsSetPagination(PageNumberPagination):
    page_size = 10
    page_size_query_param = 'page_size'
    max_page_size = 100

class PostViewSet(viewsets.ModelViewSet):
    pagination_class = StandardResultsSetPagination
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```

### Cursor Pagination

```python
from rest_framework.pagination import CursorPagination

class PostCursorPagination(CursorPagination):
    page_size = 20
    cursor_query_param = 'cursor'
    ordering = '-created_at'
```

## Optional: Filtering and Searching

### Basic Filtering

```python
from django_filters.rest_framework import DjangoFilterBackend

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    filter_backends = [DjangoFilterBackend]
    filterset_fields = ['author', 'category']
```

### Custom Filtering

```python
from rest_framework import filters

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    filter_backends = [filters.SearchFilter, filters.OrderingFilter]
    search_fields = ['title', 'content']
    ordering_fields = ['created_at', 'title']
    ordering = ['-created_at']
```

### Advanced Filtering

```python
class PostFilter(filters.FilterSet):
    min_date = filters.DateFilter(field_name="created_at", lookup_expr='gte')
    max_date = filters.DateFilter(field_name="created_at", lookup_expr='lte')

    class Meta:
        model = Post
        fields = ['author', 'category', 'min_date', 'max_date']

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    filterset_class = PostFilter
```

## Optional: Versioning

### URL Path Versioning

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.URLPathVersioning',
    'DEFAULT_VERSION': 'v1',
    'ALLOWED_VERSIONS': ['v1', 'v2'],
    'VERSION_PARAM': 'version',
}

# urls.py
router = DefaultRouter()
router.register(r'posts', PostViewSet, basename='post')

urlpatterns = [
    path('api/v1/', include(router.urls)),
    path('api/v2/', include(router.urls)),
]
```

### Custom Versioning

```python
class PostViewSet(viewsets.ModelViewSet):
    def get_serializer_class(self):
        if self.request.version == 'v2':
            return PostSerializerV2
        return PostSerializerV1
```

## Optional: Testing APIs

### Using APITestCase

```python
from rest_framework.test import APITestCase
from django.urls import reverse
from .models import Post

class PostAPITestCase(APITestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        self.post = Post.objects.create(
            title='Test Post',
            content='Test content',
            author=self.user
        )

    def test_get_post_list(self):
        url = reverse('post-list')
        response = self.client.get(url)
        self.assertEqual(response.status_code, 200)
        self.assertEqual(len(response.data), 1)

    def test_create_post(self):
        url = reverse('post-list')
        data = {
            'title': 'New Post',
            'content': 'New content'
        }
        response = self.client.post(url, data, format='json')
        self.assertEqual(response.status_code, 201)
        self.assertEqual(Post.objects.count(), 2)
```

## Best Practices

1. **Use ViewSets for standard CRUD**.
   ViewSets + routers remove repetitive boilerplate and encourage consistent endpoint behavior.
2. **Implement authentication and authorization early**.
   Decide what is public, what requires login, and what requires ownership (object-level permissions). Security is much harder to retrofit later.
3. **Add pagination to list endpoints**.
   Large result sets slow down clients and servers. Pagination keeps responses predictable and fast.
4. **Use serializers for validation**.
   Treat serializers like your API “form layer”: validate input, control output shape, and keep validation logic close to the API boundary.
5. **Add filtering/searching intentionally**.
   Filtering makes APIs usable, but it’s also a surface area for expensive queries. Add indexes and limit fields where needed.
6. **Version your APIs**.
   Versioning lets you evolve endpoints without breaking existing clients.
7. **Write endpoint tests**.
   Tests catch breaking changes and validate permissions (the most common source of subtle bugs).
8. **Use correct status codes**.
   Status codes are part of your contract. For example: 201 on create, 204 on delete, 400 on validation errors, 401/403 for auth issues.

## Summary

- **DRF transforms Django** into a powerful backend for APIs
- **Serializers convert** database models into JSON data
- **ViewSets and Routers** eliminate the need for repetitive boilerplate code
- **Permissions allow** for granular control over who can see or modify your API data
- **Authentication methods** secure your API endpoints
- **Pagination, filtering, and searching** enhance API usability

## Important Keywords

### **Django REST Framework (DRF)**

Powerful and flexible toolkit for building Web APIs on top of Django, providing serialization, authentication, and view patterns.

### **Serializer**

Component that converts complex data types (like Django models) to native Python datatypes that can then be easily rendered into JSON.

### **ModelSerializer**

Serializer class that automatically generates fields based on a Django model's structure.

### **ViewSet**

Class that combines the logic for multiple related views (list, create, retrieve, update, delete) into a single class.

### **Router**

Component that automatically generates URL patterns for ViewSets, handling all CRUD operations.

### **APIView**

Base class for building API views that provides request/response handling and authentication.

### **Generic Views**

Pre-built view classes that implement common API patterns like list-create and retrieve-update-destroy.

### **Authentication**

Process of verifying the identity of a client making API requests, using tokens, sessions, or other methods.

### **Token Authentication**

Authentication method where clients include a token in HTTP headers to identify themselves.

### **Permission**

Control mechanism that determines whether an authenticated user has access to perform specific actions on API endpoints.

### **Pagination**

Process of splitting large result sets into smaller, manageable pages for API responses.

### **Filtering**

Process of narrowing down query results based on specified criteria, improving API usability.

### **Versioning**

Practice of managing multiple versions of an API to ensure backward compatibility while evolving the API.

### **Request Object**

DRF's enhanced request object that provides access to request data, authentication, and content negotiation.

### **Response Object**

DRF's response object that handles content negotiation and proper HTTP status codes.

### **Content Negotiation**

Process of selecting the appropriate representation format (JSON, XML, etc.) based on client preferences.

### **API Documentation**

Automatically generated documentation that describes API endpoints, parameters, and response formats.

### **Status Codes**

HTTP status codes that indicate the result of an API request (200 OK, 201 Created, 404 Not Found, etc.).

### **Throttling**

Rate limiting mechanism that controls the rate of requests clients can make to prevent abuse.

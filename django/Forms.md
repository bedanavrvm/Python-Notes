
# 6. Forms: Handling User Input

In Django, Forms are a powerful tool for receiving, validating, and processing data from users. While you could write raw HTML forms, Django's Forms API automates the generation of HTML, handles security (like CSRF), and provides robust data validation.

## The Two Types of Django Forms

Django offers two primary ways to create forms depending on whether the data is going straight to a database model or not.

### 1. `forms.Form` (Standard Form)

Use this when the data is not tied to a specific database model (e.g., a "Contact Us" or "Newsletter Signup" form).

```python
from django import forms

class ContactForm(forms.Form):
    subject = forms.CharField(max_length=100)
    message = forms.CharField(widget=forms.Textarea)
    sender = forms.EmailField()
    cc_myself = forms.BooleanField(required=False)

    def clean_subject(self):
        subject = self.cleaned_data['subject']
        if 'urgent' not in subject.lower():
            raise forms.ValidationError("Please mark urgent subjects")
        return subject
```

### 2. `forms.ModelForm` (Model-Linked Form)

Use this when you want a form that directly maps to a database Model. Django will automatically generate the form fields based on the Model's fields.

```python
from django import forms
from .models import Post

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'content', 'is_draft']
        widgets = {
            'content': forms.Textarea(attrs={'rows': 10}),
            'title': forms.TextInput(attrs={'placeholder': 'Enter title'}),
        }

    def clean_title(self):
        title = self.cleaned_data['title']
        if len(title) < 5:
            raise forms.ValidationError("Title must be at least 5 characters")
        return title
```

## Form Fields and Widgets

### Common Field Types

```python
class RegistrationForm(forms.Form):
    # Text fields
    username = forms.CharField(max_length=30)
    email = forms.EmailField()
    website = forms.URLField(required=False)

    # Numeric fields
    age = forms.IntegerField(min_value=18, max_value=120)
    price = forms.DecimalField(max_digits=10, decimal_places=2)

    # Date/Time fields
    birth_date = forms.DateField(widget=forms.DateInput(attrs={'type': 'date'}))
    appointment_time = forms.TimeField(widget=forms.TimeInput(attrs={'type': 'time'}))

    # Choice fields
    gender = forms.ChoiceField(choices=[('M', 'Male'), ('F', 'Female')])
    country = forms.ChoiceField(choices=COUNTRY_CHOICES)

    # File fields
    avatar = forms.ImageField()
    document = forms.FileField()

    # Boolean
    agree_terms = forms.BooleanField(required=True)
```

### Custom Widgets

```python
class SearchForm(forms.Form):
    query = forms.CharField(
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Search...',
            'autocomplete': 'off'
        })
    )

    category = forms.ChoiceField(
        choices=[('', 'All Categories')] + CATEGORY_CHOICES,
        widget=forms.Select(attrs={'class': 'form-control'})
    )

    date_range = forms.DateField(
        widget=forms.DateInput(attrs={
            'type': 'date',
            'class': 'form-control'
        })
    )
```

## The Form Workflow in Views

Processing a form usually involves a specific "pattern" in your view to handle both the initial display (GET) and the data submission (POST).

### Basic Form Handling

```python
from django.shortcuts import render, redirect
from .forms import ContactForm

def contact_view(request):
    if request.method == 'POST':
        # 1. Bind the submitted data to a form instance
        form = ContactForm(request.POST)

        # 2. Check if the data is valid
        if form.is_valid():
            # 3. Access cleaned data
            subject = form.cleaned_data['subject']
            message = form.cleaned_data['message']
            sender = form.cleaned_data['sender']

            # Perform action (send email, etc.)
            send_contact_email(subject, message, sender)

            # Redirect after successful submission
            return redirect('contact-success')
    else:
        # 4. If GET, create a blank form
        form = ContactForm()

    return render(request, 'contact.html', {'form': form})
```

## Mini Walkthrough: Contact Form (Form → View → Template → Success Redirect)

This walkthrough connects the full workflow so you can see where each piece lives.

{% tabs %}

{% tab title="forms.py" %}

```python
from django import forms

class ContactForm(forms.Form):
    subject = forms.CharField(max_length=100)
    message = forms.CharField(widget=forms.Textarea)
    sender = forms.EmailField()
```

{% endtab %}

{% tab title="views.py" %}

```python
from django.shortcuts import redirect, render

from .forms import ContactForm

def contact_view(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            # Do something with form.cleaned_data (send email, create a ticket, etc.)
            return redirect('contact-success')
    else:
        form = ContactForm()

    return render(request, 'contact.html', {'form': form})
```

{% endtab %}

{% tab title="template" %}

```html
<!-- templates/contact.html -->
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Send</button>
</form>
```

{% endtab %}

{% endtabs %}

### ModelForm Handling

```python
from django.contrib import messages

def create_post(request):
    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES)  # Handle file uploads
        if form.is_valid():
            # Save to database but don't commit yet
            post = form.save(commit=False)
            post.author = request.user  # Add author manually
            post.save()  # Now save to database

            messages.success(request, 'Post created successfully!')
            return redirect('post-detail', pk=post.pk)
    else:
        form = PostForm()

    return render(request, 'create_post.html', {'form': form})
```

### Form Initial Data and Prefixes

```python
def edit_post(request, pk):
    post = get_object_or_404(Post, pk=pk)

    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES, instance=post)
        if form.is_valid():
            form.save()
            return redirect('post-detail', pk=post.pk)
    else:
        # Pre-populate form with existing data
        form = PostForm(instance=post)

    return render(request, 'edit_post.html', {'form': form})

# Using prefixes for multiple forms
def multi_form_view(request):
    if request.method == 'POST':
        contact_form = ContactForm(request.POST, prefix='contact')
        newsletter_form = NewsletterForm(request.POST, prefix='newsletter')

        if contact_form.is_valid() and newsletter_form.is_valid():
            # Process both forms
            pass
    else:
        contact_form = ContactForm(prefix='contact')
        newsletter_form = NewsletterForm(prefix='newsletter')

    return render(request, 'multi_form.html', {
        'contact_form': contact_form,
        'newsletter_form': newsletter_form
    })
```

## Validation and "Cleaned Data"

When you call `form.is_valid()`, Django performs several checks:

- Are all required fields present?
- Do the data types match (e.g., is an EmailField a valid email)?
- Does it pass custom validation?

If valid, the data is stored in the `form.cleaned_data` dictionary, which converts input into Python types (e.g., a string "2026-01-10" becomes a Python datetime object).

### Field-Level Validation

```python
class UserRegistrationForm(forms.Form):
    username = forms.CharField(max_length=30)
    email = forms.EmailField()
    password = forms.CharField(widget=forms.PasswordInput)
    confirm_password = forms.CharField(widget=forms.PasswordInput)

    def clean_username(self):
        username = self.cleaned_data['username']
        if User.objects.filter(username=username).exists():
            raise forms.ValidationError("Username already taken")
        return username

    def clean_email(self):
        email = self.cleaned_data['email']
        if User.objects.filter(email=email).exists():
            raise forms.ValidationError("Email already registered")
        return email

    def clean(self):
        cleaned_data = super().clean()
        password = cleaned_data.get('password')
        confirm_password = cleaned_data.get('confirm_password')

        if password and confirm_password and password != confirm_password:
            self.add_error('confirm_password', "Passwords don't match")

        return cleaned_data
```

### Custom Validators

```python
from django.core.exceptions import ValidationError
import re

def validate_phone_number(value):
    pattern = r'^\+?1?\d{9,15}$'
    if not re.match(pattern, value):
        raise ValidationError("Enter a valid phone number")

class ContactForm(forms.Form):
    phone = forms.CharField(
        validators=[validate_phone_number],
        help_text="Include country code (e.g., +1234567890)"
    )
```

## Rendering Forms in Templates

Django provides simple ways to output form fields in HTML.

### Basic Rendering

```html
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Submit</button>
</form>
```

### Rendering Options

```html
<!-- Table rendering -->
<table>
    {{ form.as_table }}
</table>

<!-- Paragraph rendering -->
{{ form.as_p }}

<!-- List rendering -->
<ul>
    {{ form.as_ul }}
</ul>
```

### Custom Form Rendering

If `{{ form.as_p }}` is too simple, you can loop through the fields manually to apply custom CSS classes.

```html
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}

    {% for field in form %}
        <div class="form-group">
            <label for="{{ field.id_for_label }}" class="form-label">
                {{ field.label }}
                {% if field.field.required %}
                    <span class="text-danger">*</span>
                {% endif %}
            </label>

            {{ field }}

            {% if field.help_text %}
                <small class="form-text text-muted">{{ field.help_text }}</small>
            {% endif %}

            {% if field.errors %}
                <div class="invalid-feedback">
                    {% for error in field.errors %}
                        <div>{{ error }}</div>
                    {% endfor %}
                </div>
            {% endif %}
        </div>
    {% endfor %}

    <button type="submit" class="btn btn-primary">Submit</button>
</form>
```

### Bootstrap 5 Form Rendering

```html
<form method="post">
    {% csrf_token %}

    {% for field in form %}
        <div class="mb-3">
            <label for="{{ field.id_for_label }}" class="form-label">
                {{ field.label }}
            </label>

            {{ field|add_class:"form-control" }}

            {% if field.errors %}
                <div class="invalid-feedback d-block">
                    {{ field.errors.0 }}
                </div>
            {% endif %}

            {% if field.help_text %}
                <div class="form-text">{{ field.help_text }}</div>
            {% endif %}
        </div>
    {% endfor %}

    <button type="submit" class="btn btn-primary">Submit</button>
</form>
```

## CSRF Protection

The `{% csrf_token %}` is a security requirement for all POST forms in Django. It prevents "Cross-Site Request Forgery" attacks by ensuring the form was submitted from your own site.

```html
<form method="post">
    {% csrf_token %}
    <!-- form fields -->
</form>
```

## Formsets

Formsets allow you to process multiple forms of the same type on a single page.

### Basic Formset

```python
from django.forms import formset_factory
from .forms import ArticleForm

# Create a formset for 3 empty forms
ArticleFormSet = formset_factory(ArticleForm, extra=3)

def manage_articles(request):
    if request.method == 'POST':
        formset = ArticleFormSet(request.POST)
        if formset.is_valid():
            for form in formset:
                if form.cleaned_data:
                    # Save each form
                    article = form.save()
            return redirect('article-list')
    else:
        formset = ArticleFormSet()

    return render(request, 'manage_articles.html', {'formset': formset})
```

### ModelFormset

```python
from django.forms.models import modelformset_factory
from .models import Book

BookFormSet = modelformset_factory(Book, fields=('title', 'author'), extra=2)

def manage_books(request):
    if request.method == 'POST':
        formset = BookFormSet(request.POST)
        if formset.is_valid():
            formset.save()
            return redirect('book-list')
    else:
        formset = BookFormSet()

    return render(request, 'manage_books.html', {'formset': formset})
```

### Formset in Templates

```html
<form method="post">
    {% csrf_token %}
    {{ formset.management_form }}

    {% for form in formset %}
        <div class="formset-row">
            {{ form }}
        </div>
    {% endfor %}

    <button type="submit">Save</button>
</form>
```

## Advanced Form Features

This section contains patterns you usually reach for once you’re comfortable with basic forms.

### File Uploads

```python
class UploadForm(forms.Form):
    file = forms.FileField()
    image = forms.ImageField()

    def clean_file(self):
        file = self.cleaned_data['file']
        if file.size > 5 * 1024 * 1024:  # 5MB limit
            raise forms.ValidationError("File too large (max 5MB)")
        return file
```

```html
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Upload</button>
</form>
```

### Dynamic Forms

```python
class DynamicForm(forms.Form):
    def __init__(self, *args, **kwargs):
        user = kwargs.pop('user', None)
        super().__init__(*args, **kwargs)

        if user and user.is_staff:
            self.fields['admin_notes'] = forms.CharField(
                widget=forms.Textarea,
                required=False
            )
```

### AJAX Form Handling

```python
from django.http import JsonResponse
from django.views.decorators.csrf import csrf_protect
from django.utils.decorators import method_decorator
from django.views import View

@method_decorator(csrf_protect, name='dispatch')
class AjaxFormView(View):
    def post(self, request):
        form = ContactForm(request.POST)
        if form.is_valid():
            # Process form
            return JsonResponse({'status': 'success'})
        else:
            return JsonResponse({
                'status': 'error',
                'errors': form.errors
            })
```

When submitting forms via JavaScript, keep CSRF protection enabled and send the token in the `X-CSRFToken` header.

## Best Practices

1. **Always use CSRF tokens** for security.
   CSRF protection prevents malicious sites from submitting forms on behalf of logged-in users. Every POST form should include `{% csrf_token %}`.
2. **Validate on the server side**.
   Client-side validation improves UX, but it’s not security. Always treat user input as untrusted and validate with forms/model validators.
3. **Use ModelForms for database-backed data**.
   ModelForms reduce duplication and ensure your form constraints match your model constraints, which prevents subtle validation mismatches.
4. **Customize error messages** for UX.
   Clear errors reduce user frustration. Prefer field-specific messages (“Title must be at least 5 characters”) over generic failures.
5. **Use form prefixes when handling multiple forms**.
   Prefixes prevent field name collisions when two forms share field names, and keep POST data unambiguous.
6. **Implement file size/type limits** for uploads.
   File uploads can become a resource and security risk. Enforce size limits and validate types, especially in public-facing apps.
7. **Pick the correct field types and widgets**.
   A good field type provides built-in validation (EmailField, URLField, DateField) and reduces custom code.
8. **Provide `help_text` and sensible defaults**.
   Help text makes forms self-explanatory and reduces support burden. Defaults and placeholders guide users toward correct input.

## Summary

- `Form` is for general data; `ModelForm` is for database records
- `is_valid()` triggers validation and prepares `cleaned_data`
- CSRF tokens are mandatory for security
- The GET/POST pattern in the View is the standard way to handle form logic
- Formsets allow handling multiple forms on one page
- Custom validation ensures data integrity

## Important Keywords

### **Form**

Django class that handles form creation, validation, and data processing for user input.

### **ModelForm**

Form class that automatically generates fields based on a Django model's structure.

### **Field**

Individual form component that represents a specific type of input data with validation rules.

### **Widget**

HTML representation of a form field, controlling how the field is rendered in the browser.

### **Validation**

Process of ensuring submitted data meets specified requirements before processing.

### **Cleaned Data**

Dictionary containing validated and type-converted form data after successful validation.

### **CSRF Token**

Security feature that prevents cross-site request forgery attacks by validating form submissions.

### **Formset**

Collection of multiple instances of the same form, allowing bulk data entry and editing.

### **Field Validation**

Validation performed on individual form fields to ensure data meets field-specific requirements.

### **Form Validation**

Validation performed on the entire form, including cross-field validation.

### **Custom Validator**

User-defined function that performs specific validation logic on form data.

### **Initial Data**

Pre-populated values provided to forms when they're first displayed.

### **Form Prefix**

String prefix used to differentiate between multiple forms on the same page.

### **File Upload**

Process of handling user-submitted files through form fields.

### **Form Rendering**

Process of displaying form fields in HTML templates using various rendering methods.

### **Error Handling**

Process of displaying validation errors to users when form data is invalid.

### **Form Media**

CSS and JavaScript files required by form widgets for proper rendering and functionality.

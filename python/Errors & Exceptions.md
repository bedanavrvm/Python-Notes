# Errors & Exceptions

In Python, errors are not just "failures"—they are objects. Managing them effectively is the difference between a program that crashes and a program that handles the unexpected gracefully. This chapter covers the mechanism for intercepting errors and creating your own.

## The Exception Lifecycle: try...except...finally

When an error occurs, Python "raises" an exception. If you don't "catch" it, the program stops. To handle this, we use a structured block of four possible parts.

Key takeaways:

- `try` is where you put code that might fail.
- `except` is where you handle specific failures.
- `else` runs only when no exception happens.
- `finally` runs no matter what (best for cleanup).

### 1. try and except

The `try` block contains code that might fail. The `except` block contains code that runs only if an error occurs.

Why/when:

- Use `try/except` when you expect a particular operation to fail sometimes (parsing user input, file access, network calls).
- Catch the most specific exceptions you can; broad catching hides bugs.

```python
try:
    number = int(input("Enter a divisor: "))
    result = 10 / number
except ZeroDivisionError:
    print("Error: You cannot divide by zero!")
except ValueError:
    print("Error: Please enter a valid integer.")

# Best Practice: Always catch specific exceptions (like ValueError) rather than using a "bare" except. A bare except can hide bugs you didn't intend to handle, like a KeyboardInterrupt.
```

### 2. else and finally

- `else`: Runs only if the `try` block was successful (no exceptions were raised)
- `finally`: Runs always, regardless of whether an error occurred or not. This is typically used for "cleanup" (e.g., closing a file or a database connection)

Key takeaway: `finally` is your last line of defense for cleanup.

```python
try:
    file = open("data.txt", "r")
    content = file.read()
except FileNotFoundError:
    print("The file does not exist.")
else:
    print("File read successfully.")
finally:
    print("Closing file resources...")

    # This check ensures we don't try to close a file that never opened
    if 'file' in locals():
        file.close()
```

### 3. Multiple except Blocks

You can catch different types of exceptions in the same try block:

Why/when:

- Use multiple `except` blocks when different failures need different handling.
- Put more specific exceptions first; broader ones later.

```python
try:
    # Some risky operation
    pass
except ZeroDivisionError:
    print("Division by zero!")
except ValueError:
    print("Invalid input!")
except TypeError:
    print("Wrong type!")
except (FileNotFoundError, PermissionError):
    print("File system error!")
except Exception as e:
    print(f"Unexpected error: {e}")
```

### 4. Exception Chaining

Python 3 allows exception chaining to preserve the original exception context:

Key takeaway: `raise ... from e` keeps the root cause visible in the traceback.

```python
try:
    # Some operation that might fail
    int("not_a_number")
except ValueError as e:
    raise RuntimeError("Invalid input") from e
```

## Raising Exceptions

Sometimes your code needs to signal that something has gone wrong based on your own logic. We use the `raise` keyword for this.

Why/when:

- Raise early when inputs violate assumptions. It is usually better than returning “sentinel” values that callers might forget to check.

```python
def set_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative!")
    print(f"Age set to {age}")
```

### Re-raising Exceptions

You can catch an exception and then re-raise it, possibly with additional context:

```python
try:
    # Some operation
    pass
except ValueError as e:
    print(f"Processing error: {e}")
    raise  # Re-raise the same exception

# Or add context (exception chaining)
try:
    # Some operation
    pass
except ValueError as e:
    raise RuntimeError("Processing failed") from e
```

## Custom Exceptions

As your application grows, built-in exceptions like `ValueError` might not be descriptive enough. You can create your own exception types by inheriting from the built-in `Exception` class.

### Creating a Custom Class

```python
class InsufficientFundsError(Exception):
    """Exception raised when a bank withdrawal exceeds balance."""

    def __init__(self, balance, amount):
        self.balance = balance
        self.amount = amount
        self.message = f"Attempted to withdraw ${amount} with only ${balance} available."
        super().__init__(self.message)

# Usage
def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientFundsError(balance, amount)
    return balance - amount

try:
    withdraw(100, 500)
except InsufficientFundsError as e:
    print(f"Transaction Failed: {e.message}")
```

### Why Use Custom Exceptions?

- **Readability**: `InsufficientFundsError` tells a developer exactly what went wrong in the business logic
- **Granular Catching**: You can catch your specific error while letting other system errors (like a `ConnectionError`) bubble up to be handled elsewhere
- **Domain-Specific**: Custom exceptions can contain domain-specific data and methods

### Multiple Inheritance

```python
class DatabaseError(Exception):
    """Base class for database errors."""
    pass

class ConnectionError(DatabaseError):
    """Database connection errors."""
    pass

class QueryError(DatabaseError):
    """Database query errors."""
    pass

# Usage
try:
    # Database operation
    pass
except ConnectionError:
    print("Cannot connect to database")
except QueryError:
    print("Invalid query")
except DatabaseError as e:
    print(f"Database error: {e}")
```

## Built-in Exception Hierarchy

Python has a well-defined exception hierarchy. Understanding it helps with catching appropriate exceptions:

Why/when:

- Catching a base class (like `OSError`) can be useful when the handling is truly the same.
- Prefer catching the specific subclass when you can (like `FileNotFoundError`).

```text
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
├── Exception
    ├── StopIteration
    ├── ArithmeticError
    │   ├── ZeroDivisionError
    │   ├── FloatingPointError
    │   └── OverflowError
    ├── AssertionError
    ├── AttributeError
    ├── LookupError
    │   ├── IndexError
    │   ├── KeyError
    │   └── ImportError
    ├── NameError
    ├── OSError
    │   ├── FileNotFoundError
    │   ├── PermissionError
    │   └── ConnectionError
    ├── RuntimeError
    ├── NotImplementedError
    ├── MemoryError
    ├── RecursionError
    ├── SyntaxError
    ├── IndentationError
    ├── TabError
    ├── SystemError
    ├── TypeError
    └── ValueError
```

### Exception Attributes

All exceptions have useful attributes:

```python
try:
    x = 1 / 0
except ZeroDivisionError as e:
    print(f"Exception type: {type(e)}")
    print(f"Exception message: {e}")
    print(f"Exception args: {e.args}")

    # Exception methods
    print(f"Has __str__: {hasattr(e, '__str__')}")
    print(f"Has __repr__: {hasattr(e, '__repr__')}")
```

## Context Managers: The with Statement

Context managers provide a clean way to handle resources that need setup and cleanup.

{% tabs %}
{% tab title="Traditional" %}
```python
file = None
try:
    file = open("data.txt", "w")
    file.write("Hello, World!")
finally:
    if file is not None:
        file.close()
```
{% endtab %}
{% tab title="Context manager" %}
```python
with open("data.txt", "w") as file:
    file.write("Hello, World!")
    # File automatically closed here
```
{% endtab %}
{% endtabs %}

### Creating Custom Context Managers

```python
class DatabaseConnection:
    def __init__(self, connection_string):
        self.connection_string = connection_string
        self.connection = None

    def __enter__(self):
        self.connection = connect_to_database(self.connection_string)
        return self.connection

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.connection:
            self.connection.close()
        if exc_type:
            print(f"Exception occurred: {exc_val}")
        return False  # Don't suppress exceptions

# Usage
with DatabaseConnection("postgresql://localhost") as db:
    # Use database connection
    data = db.query("SELECT * FROM users")
# Connection automatically closed
```

## Advanced Exception Patterns

### Exception Logging

<details>
<summary>Show exception logging example</summary>

```python
import logging

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

try:
    risky_operation()
except Exception as e:
    # Log the full exception with traceback
    logging.error("Operation failed", exc_info=True)

    # Or log specific information
    logging.warning(f"Specific error: {type(e).__name__}: {e}")
```

</details>

### Retry Mechanisms

<details>
<summary>Show retry mechanism example</summary>

```python
import time
import random

def retry_operation(max_attempts=3, delay=1):
    """Decorator to retry operations with exponential backoff."""
    def decorator(func):
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    wait_time = delay * (2 ** attempt)
                    time.sleep(wait_time + random.uniform(0, 1))
        return wrapper
    return decorator

@retry_operation(max_attempts=3)
def connect_to_server():
    # Simulated connection that might fail
    if random.random() < 0.7:
        raise ConnectionError("Connection failed")
    return "Connected"

# Usage
try:
    result = connect_to_server()
    print(f"Result: {result}")
except Exception as e:
    print(f"Failed after retries: {e}")
```

</details>

### Graceful Degradation

<details>
<summary>Show graceful degradation example</summary>

```python
class ServiceDegradationError(Exception):
    """Raised when a service is running in degraded mode."""
    pass

def get_data_with_fallback(primary_source, fallback_source):
    """Try primary source, fall back to secondary if needed."""
    try:
        return primary_source.get_data()
    except (ConnectionError, TimeoutError) as e:
        logging.warning(f"Primary source failed: {e}")
        print("Switching to fallback source")
        try:
            return fallback_source.get_data()
        except Exception as fallback_error:
            raise ServiceDegradationError(
                f"Both sources failed: Primary={e}, Fallback={fallback_error}"
            )
```

</details>

## Exception Handling Best Practices

### 1. Be Specific

{% tabs %}
{% tab title="Bad" %}
```python
try:
    operation()
except:
    pass  # Catches everything, including system exit
```
{% endtab %}
{% tab title="Good" %}
```python
try:
    operation()
except (ValueError, TypeError) as e:
    handle_specific_error(e)
except Exception as e:
    handle_unexpected_error(e)
```
{% endtab %}
{% endtabs %}

### 2. Fail Fast

{% tabs %}
{% tab title="Bad" %}
```python
def process_data(data):
    if not isinstance(data, list):
        return []  # Silent failure
    # Continue processing with empty list...
```
{% endtab %}
{% tab title="Good" %}
```python
def process_data(data):
    if not isinstance(data, list):
        raise TypeError(f"Expected list, got {type(data).__name__}")
```
{% endtab %}
{% endtabs %}

### 3. Don't Suppress Important Errors

{% tabs %}
{% tab title="Bad" %}
```python
try:
    config = load_config()
except:
    config = {}  # Use default config, but problem might persist
```
{% endtab %}
{% tab title="Good" %}
```python
try:
    config = load_config()
except FileNotFoundError:
    print("Configuration file missing!")
    raise  # Re-raise for caller to handle
```
{% endtab %}
{% endtabs %}

### 4. Provide Context

{% tabs %}
{% tab title="Bad" %}
```python
raise ValueError("Error in processing")
```
{% endtab %}
{% tab title="Good" %}
```python
raise ValueError(
    f"Cannot process user {user_id}: "
    f"Invalid age {age}. Age must be between 0 and 120."
)
```
{% endtab %}
{% endtabs %}

### 5. Use Type Hints

```python
from typing import Union, Optional

def process_user_data(
    user_id: int,
    data: dict,
    required_field: str
) -> Optional[str]:
    """
    Process user data and return specific field.

    Raises:
        KeyError: If required field is missing
        TypeError: If data is not a dictionary
    """
    if not isinstance(data, dict):
        raise TypeError("Data must be a dictionary")

    try:
        return data[required_field]
    except KeyError:
        raise KeyError(f"Missing required field: {required_field}")
```

## Debugging Exceptions

### Exception Tracing

<details>
<summary>Show exception tracing helper</summary>

```python
import traceback
import sys

def debug_exceptions(func):
    """Decorator to print full traceback on exception."""
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except Exception as e:
            print(f"Exception in {func.__name__}:")
            traceback.print_exc(file=sys.stderr)
            raise
    return wrapper

@debug_exceptions
def buggy_function():
    return 1 / 0
```

</details>

### Post-Mortem Analysis

<details>
<summary>Show post-mortem analysis hook</summary>

```python
import traceback
import sys

def exception_summary(exc_type, value, tb):
    """Create a summary of exception information."""
    return {
        'type': exc_type.__name__,
        'message': str(value),
        'file': tb.tb_frame.f_code.co_filename,
        'line': tb.tb_frame.f_lineno,
        'function': tb.tb_frame.f_code.co_name
    }

# Global exception hook
def handle_unhandled_exception(exc_type, value, tb):
    summary = exception_summary(exc_type, value, tb)
    print("Unhandled Exception:")
    for key, val in summary.items():
        print(f"  {key}: {val}")

sys.excepthook = handle_unhandled_exception
```

</details>

## Summary

- `try`: Run code that might fail
- `except`: Handle specific errors
- `else`: Run if no errors occurred
- `finally`: Run no matter what (cleanup)
- `raise`: Manually trigger an exception
- Custom Exceptions: Inherit from `Exception` class to create domain-specific error types

## Important Keywords

### **Exception**

An event that occurs during program execution that disrupts normal flow. Python uses objects to represent errors rather than error codes.

### **try-except**

Control structure for catching and handling exceptions, separating normal flow from error handling.

### **finally**

Code block that executes regardless of whether an exception occurred, typically used for cleanup operations.

### **raise**

Statement to manually trigger an exception, used for error signaling in custom logic.

### **Custom Exception**

User-defined exception class that inherits from `Exception` or its subclasses, providing domain-specific error handling.

### **Exception Hierarchy**

Tree-like structure of exception classes showing inheritance relationships, with `BaseException` at the root.

### **Exception Chaining**

Technique to link multiple exceptions together, preserving original exception context while adding new information.

### **Context Manager**

Object that manages resource setup and cleanup automatically, used with `with` statements.

### **Exception Handling**

Process of responding to exceptions, including catching, logging, retrying, and graceful degradation.

### **Error Propagation**

Mechanism by which exceptions bubble up through call stack until caught by appropriate handler.

### **Traceback**

Detailed information about exception's origin, including file names, line numbers, and call stack.

### **Graceful Degradation**

Strategy for handling failures by falling back to alternative functionality rather than complete failure.

### **Fail Fast**

Principle of aborting operation immediately when invalid state is detected, rather than continuing with corrupted data.

### **Logging**

Recording of exception information for debugging and monitoring purposes.

### **Retry Mechanism**

Pattern for automatically reattempting failed operations with delays between attempts.

### **Exception Safety**

Design approach that ensures program continues operating correctly even when unexpected errors occur.

### **Unhandled Exception**

Exception that reaches the top level of a program without being caught, potentially causing crashes.

### **Error Recovery**

Process of restoring program state after an error occurs, allowing continued operation.

### **Exception Attributes**

Properties and methods available on exception objects, such as `args`, `message`, and traceback information.

### **Type Safety**

Practice of ensuring operations are performed on appropriate data types, preventing `TypeError` exceptions.

### **Validation**

Process of checking inputs and state before operations, raising exceptions for invalid conditions.

### **Resource Management**

Proper handling of system resources like files, network connections, and memory to prevent leaks.

### **Exception Handling Patterns**

Standardized approaches for dealing with common error scenarios across different applications and domains.

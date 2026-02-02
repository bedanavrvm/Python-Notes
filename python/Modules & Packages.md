# Modules and Packages

As your Python projects grow, keeping all your code in a single file becomes impossible to manage. Python provides a robust system for modularization, allowing you to split code into logical units that can be reused across different projects.

## Modules: The Single File

A Module is simply a file containing Python definitions and statements. The file name is the module name with the suffix `.py` appended.

### The import System

When you import a module, Python executes the code in that file and creates a module object in the current scope.

This tabbed example compares the most common import styles:
- `import module` keeps the module namespace (you access `module.name`).
- `from module import name` pulls specific names into your local scope.
- The trade-off is clarity vs. brevity: explicit module prefixes make large codebases easier to read.

{% tabs %}
{% tab title="calculator.py" %}
```python
def add(a, b):
    return a + b
```
{% endtab %}
{% tab title="main.py (import module)" %}
```python
import calculator
print(calculator.add(5, 3))
```
{% endtab %}
{% tab title="main.py (from ... import ...)" %}
```python
from calculator import add
print(add(5, 3))
```
{% endtab %}
{% endtabs %}

### Module Search Path (`sys.path`)

When you run `import abc`, Python looks for a file named `abc.py` in this order:

1. The directory containing the input script (current directory)
2. `PYTHONPATH` (an environment variable)
3. The installation-dependent default (standard libraries)

This snippet prints `sys.path` so you can see where Python is looking.
It is useful when imports work in one environment but fail in another.

```python
import sys
print("Module search paths:")
for path in sys.path:
    print(f"  {path}")
```

### Module Attributes

Every module has special attributes that provide information about the module:

These attributes are commonly used for debugging, logging, and tooling:
- `__name__`: the module's import name.
- `__file__`: where it was loaded from.
- `__dict__`: the module namespace (everything defined at top-level).

```python
# calculator.py
"""
Calculator module for basic arithmetic operations.
Author: Your Name
Version: 1.0
"""

def add(a, b):
    return a + b

# Access module attributes
import calculator
print(calculator.__name__)    # 'calculator'
print(calculator.__doc__)     # Module docstring
print(calculator.__file__)    # Path to module file
print(calculator.__dict__)    # Module's symbol table
```

### Module Reloading

During development, you can reload modules to pick up changes without restarting:

Reloading is mostly a REPL/notebook convenience. In production systems, you
typically restart the process instead of reloading modules at runtime.

```python
import importlib
import calculator

# Make changes to calculator.py
# Then reload
importlib.reload(calculator)
```

### Module Cache

Python caches compiled bytecode in `__pycache__` directories to speed up
subsequent imports. This **module cache** avoids recompiling unchanged modules.

## Packages: The Directory Structure

A Package is a way of structuring Python's module namespace by using "dotted module names." A package is essentially a folder containing multiple module files.

### The Role of `__init__.py`

For a folder to be treated as a package, it traditionally required a file named `__init__.py`.

- In modern Python (3.3+), this file is optional (Namespace Packages), but it is still used to perform package-level initialization.
- Code inside `__init__.py` runs automatically when the package is imported.

### Example Structure

```text
my_project/
├── main.py
└── graphics/                # This is a package
    ├── __init__.py
    ├── primitives.py        # This is a module
    └── effects.py          # This is a module
```

### Creating Packages

This example shows a few important package patterns:
- Relative imports like `from .primitives import draw_line` only work when imported as a package.
- `__all__` is an *export list* that controls what `from package import *` exposes.
- `__version__` is a common convention for package version metadata.

```python
# graphics/__init__.py
"""
Graphics package for drawing operations.
"""

# Package-level imports
from .primitives import draw_line
from .effects import apply_blur

# Package version
__version__ = "1.0.0"
__all__ = ["draw_line", "apply_blur"]  # Control what gets imported with *
```

## Absolute vs. Relative Imports

When working within a package, you have two ways to reference other modules.

### 1. Absolute Imports

Specifies the full path from the project's root folder. This is the preferred method (PEP 8) because it is clear and unambiguous.

{% tabs %}
{% tab title="Absolute" %}
```python
# Inside effects.py
from graphics.primitives import draw_line
```
{% endtab %}
{% tab title="Relative" %}
```python
# Inside effects.py
from .primitives import draw_line

# Warning: Relative imports only work within packages.
# If you try to run effects.py as a standalone script,
# relative import will fail.
```
{% endtab %}
{% endtabs %}

### 2. Relative Imports

Uses leading dots to indicate the current and parent packages.

- `.` refers to the current package
- `..` refers to the parent package

### Import Best Practices

A few practical rules that keep imports predictable and readable:
- Keep imports at the top of the file.
- Prefer explicit imports over wildcard imports.
- Use aliases (`import numpy as np`) when the alias is conventional.
- Avoid multiple imports on one line to keep diffs clean.

```python
# Good: Specific imports
from os import path
from json import loads

# Avoid: Wildcard imports (except in specific cases)
# from graphics import *  # Can pollute namespace

# Use aliases for clarity
import numpy as np
import pandas as pd

# Group related imports
import os
import sys
from pathlib import Path
```

## Executing Modules as Scripts: `if __name__ == "__main__"`

Every Python module has a built-in variable called `__name__`.

- If the file is being run directly, `__name__` is set to `"__main__"`
- If the file is being imported, `__name__` is set to the filename

This allows you to write code that only runs when the file is executed as a script, but not when it is imported as a library.

```python
def main():
    print("This script is running directly!")

if __name__ == "__main__":
    main()
```

## Standard Library Structure

Python follows the "Batteries Included" philosophy. Some essential modules you should know:

### `os` / `pathlib`

Interacting with the operating system and file paths.

This example uses `pathlib` for path handling and `os` for OS-level info.

```python
import os
from pathlib import Path

# Create directory
Path("my_folder").mkdir(exist_ok=True)

# List files
for file in Path(".").iterdir():
    print(file)

# Get current working directory
print(os.getcwd())
```

### `sys`

System-specific parameters and functions.

`sys.argv` is how scripts receive command line arguments. `sys.exit()` is a
conventional way to return a success/failure exit code to the shell.

```python
import sys

# Command line arguments
print(sys.argv)

# Exit program
sys.exit(0)

# Platform information
print(sys.platform)
```

### `json`

Parsing and creating JSON data.

`json.dumps()` turns Python objects into JSON text.
`json.loads()` parses JSON text back into Python objects.

```python
import json

data = {"name": "Alice", "age": 25}
json_string = json.dumps(data, indent=2)

# Parse JSON
parsed = json.loads(json_string)
print(parsed["name"])  # Alice
```

### `datetime`

Handling dates and times.

`datetime` supports arithmetic with `timedelta`, which is usually easier and
safer than manually calculating seconds.

```python
from datetime import datetime, timedelta

now = datetime.now()
future = now + timedelta(days=7)

print(f"Current: {now}")
print(f"Future: {future}")
```

### `math`

Mathematical constants and functions.

Use `math` for numeric primitives; for large array math, you typically move
to `numpy`.

```python
import math

print(math.pi)      # 3.14159...
print(math.sqrt(16)) # 4.0
print(math.factorial(5)) # 120
```

## Advanced Module Concepts

These topics matter more as projects grow:

- You start shipping code as packages.
- Imports become part of your architecture (boundaries, plugin systems).
- Debugging import behavior becomes a practical skill.

### Namespace Packages

Modern Python allows packages without `__init__.py` files:

Why/when:

- Namespace packages are most common when multiple distributions contribute
  to the same package namespace.
- In many projects, a regular package with `__init__.py` is simpler and more
  explicit.

```text
# my_project/
├── main.py
└── graphics/
    ├── primitives.py
    └── effects.py

# graphics/__init__.py is optional or empty
# Python 3.3+ automatically treats it as a namespace package
```

### Circular Imports

When two modules import each other, you can create circular dependencies:

Key takeaway: circular imports are usually a design smell. If two modules need
each other, consider extracting shared code into a third module.

```python
# module_a.py
import module_b
def func_a():
    return module_b.func_b()

# module_b.py
import module_a
def func_b():
    return module_a.func_a()

# This creates a circular import
# Solution: restructure or use lazy imports
```

### Dynamic Imports

Import modules dynamically at runtime:

Why/when:

- Plugin systems and optional dependencies.
- Avoiding heavy imports on startup.
- Loading modules by name from configuration.

```python
import importlib
import importlib.util

# Import by name
module = importlib.import_module("math")
result = module.sqrt(16)

# Import from file path
spec = importlib.util.spec_from_file_location("custom_module", "/path/to/module.py")
if spec is None or spec.loader is None:
    raise ImportError("Could not load module spec")
custom_module = importlib.util.module_from_spec(spec)
spec.loader.exec_module(custom_module)
```

### Lazy Import

A **lazy import** defers loading until the module is actually used, improving
startup performance:

Key takeaway: lazy imports trade a small delay at first use for faster startup
and fewer optional dependency issues.

```python
def heavy_operation():
    import pandas as pd  # Imported only when function is called
    return pd.DataFrame()
```

### Package Distribution

Creating distributable packages:

Why/when:

- Use packaging when you want reuse across repositories or want to publish to
  an internal index/PyPI.
- Modern projects usually prefer `pyproject.toml`, but `setup.py` is still
  widely understood.

<details>
<summary>Show setup.py example</summary>

```python
# setup.py
from setuptools import setup

setup(
    name="my_package",
    version="1.0.0",
    description="A sample Python package",
    author="Your Name",
    packages=["my_package"],
    install_requires=["requests>=2.0"],
    python_requires=">=3.6"
)
```

</details>

#### Package Metadata

**Package metadata** includes name, version, author, and dependencies, stored in
`setup.py` or `pyproject.toml`. Tools like `pip` read this metadata.

#### Entry Point

An **entry point** is a script or function defined in package metadata that can
be invoked via command-line tools (e.g., `console_scripts` in `setup.cfg`).

### Virtual Environments

Isolating package dependencies:

Key takeaway: a virtual environment isolates dependencies per project, which
prevents version conflicts.

{% tabs %}
{% tab title="Linux/macOS" %}
```bash
# Create virtual environment
python -m venv my_env

# Activate
source my_env/bin/activate

# Install packages
pip install requests numpy pandas

# Freeze dependencies
pip freeze > requirements.txt
```
{% endtab %}
{% tab title="Windows" %}
```bash
# Create virtual environment
python -m venv my_env

# Activate
my_env\Scripts\activate

# Install packages
pip install requests numpy pandas

# Freeze dependencies
pip freeze > requirements.txt
```
{% endtab %}
{% endtabs %}

#### Version Management

**Version management** involves handling package versions and ensuring
compatibility. Use `pip-tools` or `poetry` to lock dependency versions.

#### Semantic Versioning

**Semantic versioning** (MAJOR.MINOR.PATCH) conveys API compatibility:

- MAJOR: incompatible changes
- MINOR: new features, backward compatible
- PATCH: bug fixes

## Module Discovery and Inspection

Why/when:

- Useful for debugging and tooling.
- Useful for building developer-friendly CLIs and plugin systems.

### Inspecting Modules

Key takeaways:

- `inspect.signature()` is a great way to print a callable's parameters.
- `inspect.getsource()` only works when source code is available.

```python
import inspect
import calculator

# Get module info
print(inspect.getsource(calculator.add))
print(inspect.getdoc(calculator.add))
print(inspect.signature(calculator.add))

# List all functions
for name, obj in inspect.getmembers(calculator):
    if inspect.isfunction(obj):
        print(f"Function: {name}")
```

### Package Resources

Accessing data files within packages:

Why/when:

- Packages often include non-code assets (JSON, templates, SQL, model files).
- Prefer modern `importlib.resources` for new code; `pkg_resources` is older
  and heavier, but still common.

<details>
<summary>Show package resources example</summary>

```python
# my_package/
# ├── data/
# │   └── config.json
# └── __init__.py

# In __init__.py
import pkg_resources
import json

def load_config():
    with pkg_resources.resource_stream("my_package", "data/config.json") as f:
        return json.load(f)
```

</details>

### Import Hooks

An **import hook** allows customizing the import process by registering finders
and loaders. This enables loading modules from databases, zip files, or remote
sources.

Key takeaway: import hooks are powerful and advanced—use them when you truly
need custom loading behavior.

### Plugin Systems

A **plugin system** loads additional functionality dynamically at runtime.
Plugins are modules that implement a defined interface and are discovered via
entry points or directory scanning.

Why/when:

- Extensible applications (formatters, linters, web frameworks).
- When you want third-party code to register functionality without modifying
  the core codebase.

### Library vs Framework

- **Library**: Code you call (e.g., `requests`, `numpy`). You control the flow.
- **Framework**: Code that calls you (e.g., Django, Flask). The framework
  controls the flow via inversion of control.

### API Design

Good **API design** for modules means:

- Clear function signatures and docstrings
- Consistent naming conventions
- Minimal, stable public interfaces
- Use of `__all__` to control what gets imported with `*`

### Package Index

A **package index** like PyPI hosts Python packages. `pip install` fetches
packages from the configured index (default: <https://pypi.org/simple>).

## Summary

- Modules are `.py` files; Packages are directories containing modules
- `__init__.py` marks a directory as a package and handles initialization
- Absolute imports are the standard for professional Python development
- `if __name__ == "__main__"` prevents code from running during an import

## Important Keywords

### **Module**

Single file containing Python definitions, statements, and functions that can be imported and used in other programs.

### **Package**

Directory containing multiple Python modules, organized with `__init__.py` file to mark it as a package.

### **Import**

Mechanism for loading code from modules into the current namespace, enabling code reuse and organization.

### **Module Object**

Object created when a module is imported, containing the module's namespace and attributes.

### **Namespace**

Container that holds the names (variables, functions, classes) defined in a module or package.

### **`__init__.py`**

Special file that marks a directory as a Python package and can contain initialization code.

### **Absolute Import**

Import statement specifying the full path from the project root, providing clear and unambiguous module references.

### **Relative Import**

Import using dot notation to reference modules within the same package hierarchy.

### **sys.path**

List of directories Python searches when importing modules, determining where to find module files.

### **PYTHONPATH**

Environment variable that adds additional directories to Python's module search path.

### **`__name__`**

Special variable that indicates whether a module is being run as a script or imported as a library.

### **`__main__`**

Value of `__name__` when a Python file is executed directly, allowing script-specific code execution.

### **Module Reloading**

Process of reloading a previously imported module to pick up changes without restarting the Python interpreter.

### **Wildcard Import**

Import statement using `*` to import all public names from a module, potentially causing namespace pollution.

### **Circular Import**

Situation where two or more modules import each other, creating a dependency cycle that must be resolved.

### **Dynamic Import**

Importing modules programmatically at runtime using functions like `importlib.import_module()`.

### **Package Distribution**

Process of packaging Python code for distribution, typically using tools like setuptools or poetry.

### **Virtual Environment**

Isolated Python environment with its own installed packages and dependencies, preventing conflicts between projects.

### **Module Inspection**

Process of examining module contents, functions, and metadata programmatically using the inspect module.

### **Namespace Package**

Modern Python package format that doesn't require `__init__.py` files, treating directories as packages through namespace mechanisms.

### **Resource Management**

Technique for accessing non-code files (data, templates, configuration) within packages using importlib resources.

### **Standard Library**

Collection of built-in modules included with Python, providing core functionality without external dependencies.

### **Third-Party Package**

External Python library distributed through package managers like PyPI, extending Python's capabilities beyond the standard library.

### **Import Hook**

Mechanism allowing customization of the import process by registering functions that get called during module loading.

### **Module Cache**

Python's internal caching of compiled bytecode to speed up subsequent imports of the same module.

### **Package Metadata**

Information about a package such as version, author, dependencies, typically stored in setup.py or pyproject.toml.

### **Entry Point**

Specific function or script that serves as the main execution interface for a package or application.

### **Lazy Import**

Technique of deferring module loading until the module is actually used, improving startup performance.

### **Plugin System**

Architecture where additional functionality is loaded dynamically through modules or packages at runtime.

### **Library vs Framework**

Distinction between reusable code collections (libraries) and applications that provide structure and control flow (frameworks).

### **API Design**

Practice of designing clear interfaces for modules and packages, specifying what functions are available and how to use them.

### **Version Management**

Process of handling different versions of packages and ensuring compatibility across development and deployment environments.

### **Semantic Versioning**

Version numbering scheme that conveys meaning about changes (major.minor.patch) following established conventions.

### **Package Index**

Repository like PyPI where Python packages are published and can be discovered and installed using tools like pip.

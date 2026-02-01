# Introduction to Python

In this chapter, you'll get a high-level view of what Python is, how it
runs your code under the hood, and how to set up a clean, professional
development environment. By the end, you should be able to install
Python, create an isolated project, and explain the basics of how the
interpreter executes scripts.

## What is Python?

Python is a high-level, interpreted, object-oriented programming language
with dynamic semantics. Its built-in data types, expressive syntax, and
rich standard library make it productive for scripting, automation, web
development, and rapid application development. Python emphasizes
readability, which keeps programs approachable as they grow.

### Key Characteristics

- **Dynamic typing:** Names are bound to objects at runtime, so you do not
  declare types up front.
- **Automatic memory management:** Reference counting and a cyclic garbage
  collector handle allocation and cleanup.
- **Multi-paradigm:** Use procedural, object-oriented, or functional
  styles depending on the problem.
- **Extensive standard library:** The "batteries included" modules cover
  files, networking, testing, and more.
- **Cross-platform:** The same code runs on Windows, macOS, Linux, and
  most cloud runtimes.

#### History and Philosophy

Created by Guido van Rossum and first released in 1991, Python was
designed with an emphasis on code readability and a syntax that lets
programmers express ideas in fewer lines. The name is a nod to the
British comedy group Monty Python, reflecting the community's playful
spirit.

### Interpreted vs. Compiled

To understand Python, you must understand how it executes code compared
to languages like C++ or Rust.

- **Compiled languages (e.g., C++):** Source code is translated into
  machine code ahead of time. This yields fast execution but a slower,
  compile-first workflow.
- **Interpreted languages (e.g., Python):** Source code is executed by an
  interpreter at runtime, enabling faster iteration and easier
  debugging.

**The hybrid reality:** CPython first compiles source code to bytecode
(`.pyc` files stored in `__pycache__`) and then executes that bytecode
with the Python Virtual Machine (PVM). This keeps the rapid edit-run
cycle while still caching work between runs.

#### Performance Considerations

- **JIT compilation:** Alternative runtimes like PyPy use JITs to speed
  up long-running programs (CPython does not).
- **Bytecode caching:** `.pyc` files avoid recompilation on subsequent
  runs.
- **Performance vs. development speed:** Python favors developer
  productivity, but you can optimize hotspots with profiling or native
  extensions when needed.

### Setting up the Environment

A professional Python workflow avoids installing libraries globally.
Instead, you isolate dependencies per project so tools do not conflict
between apps. Virtual environments provide that sandbox.

#### 1. The Interpreter

First, ensure you have Python installed. You can check this in your terminal:

{% tabs %}

{% tab title="python" %}

```bash
python --version
```

{% endtab %}

{% tab title="python3" %}

```bash
python3 --version
```

{% endtab %}

{% endtabs %}

If `python` is not found on Windows, try the `py` launcher
(`py --version`).

#### 2. Virtual Environments (venv)

A virtual environment is a self-contained directory tree that contains
a Python installation for a particular version of Python, plus a number
of additional packages.

**Why use them?** If Project A needs Django 3.0 and Project B needs
Django 4.0, installing them globally would cause conflicts. Virtual
environments keep each project's dependencies isolated.

**Creation and Activation:**

1. Create the environment (named `venv`):

```bash
python -m venv venv
```

2. Activate it:

{% tabs %}

{% tab title="Windows" %}

```bash
.\venv\Scripts\activate
```

{% endtab %}

{% tab title="macOS/Linux" %}

```bash
source venv/bin/activate
```

{% endtab %}

{% endtabs %}

Once activated, your terminal prompt will usually show `(venv)`,
indicating that any Python commands now stay within this bubble. Run
`deactivate` to exit the environment.

### 3. PIP (Package Installer for Python)

PIP is the standard package manager. It installs, upgrades, and removes
packages from the Python Package Index (PyPI). When multiple Python
versions are installed, prefer `python -m pip` to target the active
interpreter.

```bash
# Install a package
python -m pip install requests

# List installed packages
python -m pip list

# Create a requirements file (for sharing projects)
python -m pip freeze > requirements.txt

# Install from a requirements file
python -m pip install -r requirements.txt

# Upgrade a package
python -m pip install --upgrade package_name

# Uninstall a package
python -m pip uninstall package_name
```

#### Alternative Package Managers

- **Conda:** Popular in data science; manages Python packages and system
  dependencies.
- **Poetry:** Modern dependency management and packaging with lockfiles.
- **Pipenv:** Combines pip and virtualenv functionality in a single tool.

## Hello, World

The simplest Python program is a single line. Unlike Java or C++,
Python does not require a class wrapper or a main function header
for simple scripts.

```python
print("Hello, World!")
```

Save it as `hello.py` and run it from the terminal.

### Running Python Code

Python offers an interactive REPL for quick experiments and a script
runner for full programs:

```bash
## Interactive mode (REPL)
python
>>> print("Hello, World!")
Hello, World!
>>> exit()

## Running a script file
python hello.py

## Running with specific Python version
python3 hello.py
```

#### Python File Extensions

- `.py`: Standard Python source files
- `.pyc`: Compiled bytecode files (auto-generated)
- `.pyi`: Stub files for type hints
- `.pyw`: Windows GUI applications (no console window)
- `.pyz`: Zip archive containing Python modules

## The "Pythonic" Philosophy (PEP 8)

In the JavaScript world, you have "clean code." In Python, we talk about
"Pythonic" code: code that aligns with the Zen of Python (PEP 20) and the
style guidance in PEP 8.

You can see the philosophy yourself by running:

```python
import this
```

### Core Tenets

- Beautiful is better than ugly.
- Explicit is better than implicit.
- Simple is better than complex.
- Complex is better than complicated.
- Flat is better than nested.
- Sparse is better than dense.
- Readability counts.
- Special cases aren't special enough to break the rules.
- Although practicality beats purity.
- Errors should never pass silently.
- Unless explicitly silenced.
- In the face of ambiguity, refuse the temptation to guess.
- There should be one-- and preferably only one --obvious way to do it.
- Although that way may not be obvious at first unless you're Dutch.
- Now is better than never.
- Although never is often better than *right* now.

### PEP 8 – The Style Guide

PEP 8 is the official document that defines how Python code should be formatted.

| Feature | PEP 8 Standard |
|---------|----------------|
| Indentation | Use 4 spaces per indentation level (no tabs). |
| Variable Names | Use snake_case (e.g., my_variable). |
| Class Names | Use PascalCase (e.g., MyClass). |
| Function Names | Use snake_case (e.g., calculate_sum). |
| Constants | Use UPPER_SNAKE_CASE (e.g., MAX_RETRIES). |
| Private Members | Prefix with underscore (e.g., _internal_method). |
| Line Length | Limit all lines to a maximum of 79 characters. |
| Blank Lines | Surround top-level function and class definitions with two blank lines.|
| Imports | Place imports at the top of the file, one per line. |
| Docstrings | Use triple quotes for module, class, and function documentation. |
| Spacing | Use spaces around operators and after commas. |

### Example of Non-Pythonic vs. Pythonic code

{% tabs %}

{% tab title="Non-Pythonic" %}

```python
# ❌ Non-Pythonic (unclear names, cramped formatting)
def check_val(X):
    if X > 10: return True
    else: return False
```

{% endtab %}

{% tab title="Pythonic" %}

```python
# ✅ Pythonic (Readable, follows PEP 8)
def is_threshold_reached(value):
    """Checks if the value exceeds the standard threshold."""
    return value > 10
```

{% endtab %}

{% endtabs %}

## Python Versions and Compatibility

### Python 2 vs Python 3

Python 3 was released in 2008 and is not backward-compatible with
Python 2. Python 2 reached end-of-life in 2020 and is no longer
supported. All new development should target Python 3.

#### Key Differences

- **Print Statement:** `print "Hello"` (Python 2) vs `print("Hello")` (Python 3)
- **Integer Division:** `5/2 = 2` (Python 2) vs `5/2 = 2.5` (Python 3)
- **Unicode:** Strings are Unicode by default in Python 3
- **xrange:** `xrange()` (Python 2) vs `range()` (Python 3)

#### Current Python 3 Versions

- **3.12:** Latest stable release (late 2023; check python.org for updates)
- **3.11:** Performance-focused release with significant speed improvements
- **3.10:** Introduced structural pattern matching and improved error messages

### Choosing a Python Version

- Use the latest stable version for new projects
- Consider legacy dependencies when maintaining older codebases
- Pin the version in tooling or CI to keep environments consistent
- Some scientific computing packages may lag behind the latest releases

## Python Implementations

### CPython

- The standard and most widely used implementation
- Written in C and Python
- Reference implementation that other versions are based on

### Alternative Implementations

- **PyPy:** JIT-compiled implementation, often faster
  for long-running applications
- **Jython:** Runs on the Java Virtual Machine
- **IronPython:** Runs on the .NET Framework
- **MicroPython:** Optimized for microcontrollers
  and embedded systems

Most third-party packages target CPython first, so verify compatibility
when using alternative runtimes.

## Summary

- Python executes bytecode on the PVM, balancing fast iteration with
  caching between runs.
- Use virtual environments to isolate dependencies and avoid conflicts.
- The Zen of Python (PEP 20) and PEP 8 emphasize readability and
  explicitness.
- Python 3 is the current standard; Python 2 is no longer supported.
- CPython is the default implementation, but alternatives exist for
  specific performance or platform needs.
- The language's philosophy emphasizes simplicity, readability, and
  explicit design.

## Important Keywords

### **Bytecode**

Intermediate instructions (`.pyc` files stored in `__pycache__`) generated
from Python source and executed by the Python Virtual Machine (PVM).

### **Dynamic Typing**

Types belong to objects and are determined at runtime. Type hints can be
added for clarity and tooling, but they are optional.

### **Garbage Collection**

Automatic memory management in CPython using reference counting plus a
cycle detector to reclaim unused objects.

### **REPL (Read-Eval-Print Loop)**

Interactive Python shell for rapid experiments and debugging.

### **PyPI (Python Package Index)**

Official repository for Python packages; accessed via pip.

### **Virtual Environment**

Isolated environment with its own interpreter and site-packages,
preventing dependency conflicts between projects.

### **JIT Compilation (Just-In-Time)**

Compilation during execution; used by runtimes like PyPy to speed up
long-running programs.

### **Type Hints**

Optional annotations (PEP 484) that document expected types and enable
static analysis tools.

### **PEP (Python Enhancement Proposal)**

Numbered design documents that define new features or community
guidelines (for example, PEP 8 and PEP 20).

### **Multi-paradigm**

Support for multiple programming approaches: procedural (step-by-step),
object-oriented (objects and classes), and functional (pure functions).

### **Cross-platform**

Ability to run on multiple operating systems (Windows, macOS, Linux)
without code modification.

### **Standard Library**

Collection of modules included with Python installation, providing the
"batteries included" functionality for common tasks.

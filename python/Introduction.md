# Introduction to Python

In this chapter, we will discuss what Python is, how it functions
under the hood, and how to set up a professional development environment.

## What is Python?

Python is a high-level, interpreted, object-oriented programming
language with dynamic semantics. Its high-level built-in data structures,
combined with dynamic typing and dynamic binding, make it very attractive
for Rapid Application Development.

### Key Characteristics

- **Dynamic Typing:** Variable types are determined at runtime,
  not declared in advance
- **Memory Management:** Automatic garbage collection handles
  memory allocation and deallocation
- **Multi-paradigm:** Supports procedural, object-oriented,
  and functional programming styles
- **Extensive Standard Library:** "Batteries included" philosophy
  with rich built-in modules
- **Cross-platform:** Runs on Windows, macOS, Linux,
  and many other systems

#### History and Philosophy

Created by Guido van Rossum and first released in 1991,
Python was designed with emphasis on code readability and
syntax that allows programmers to express concepts in fewer lines
of code. The language is named after the British comedy group
Monty Python.

### Interpreted vs. Compiled

To understand Python, you must understand how it executes code
compared to languages like C++ or Rust.

- **Compiled Languages (e.g., C++):** The source code is transformed into
  machine code (binary) by a compiler before the program runs.
  This results in very fast execution but a slower development cycle.
- **Interpreted Languages (e.g., Python):** The code is processed at
  runtime by the interpreter. There is no separate compilation step
  required by the user.

**The Hybrid Reality:** Technically, Python is "Byte-compiled."
  When you run a script, Python first compiles your source code into
  an intermediate format called Bytecode (.pyc files).
  This Bytecode is then executed by the Python Virtual Machine (PVM).

#### Performance Considerations

- **Just-In-Time (JIT) Compilation:** Modern Python implementations
  like PyPy use JIT compilation to improve performance
- **Caching:** Bytecode is cached in `.pyc` files
  to avoid recompilation on subsequent runs
- **Performance vs. Development Speed:** Python trades raw execution
  speed for faster development and easier debugging

### Setting up the Environment

A professional Python workflow avoids installing libraries globally.
Instead, we use "sandboxes" for each project.

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

#### 2. Virtual Environments (venv)

A virtual environment is a self-contained directory tree that contains
a Python installation for a particular version of Python, plus a number
of additional packages.

**Why use them?** If Project A needs Django 3.0 and Project B needs
  Django 4.0, installing them globally would cause a conflict.
  Virtual environments solve this.

**Creation and Activation:**

```bash
## 1. Create the environment (named 'venv')
python -m venv venv
```

## 2. Activate it

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

Once activated, your terminal prompt will usually show (venv),
indicating that any Python commands now stay within this bubble.

### 3. PIP (Package Installer for Python)

PIP is the standard package manager. It allows you to install,
upgrade, and remove packages from the Python Package Index (PyPI).

```bash
## Installing a package
pip install requests

## Listing installed packages
pip list

## Creating a requirements file (for sharing projects)
pip freeze > requirements.txt

## Installing from requirements file
pip install -r requirements.txt

## Upgrading a package
pip install --upgrade package_name

## Uninstalling a package
pip uninstall package_name
```

#### Alternative Package Managers

- **Conda:** Popular in data science, manages both Python packages
  and system dependencies
- **Poetry:** Modern dependency management and packaging tool
  with better dependency resolution
- **Pipenv:** Combines pip and virtualenv functionality
  in a single tool

## Hello, World

The simplest Python program is a single line. Unlike Java or C++,
Python does not require a class wrapper or a main function header
for simple scripts.

```python
print("Hello, World!")
```

### Running Python Code

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

In the JavaScript world, you have "clean code." In Python, we have
"Pythonic" code. This refers to code that follows the Zen of Python
(PEP 20).

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
## ❌ Non-Pythonic (Messy, confusing names)
def check_val(X):
 if X > 10:return True
 else:return False
```

{% endtab %}

{% tab title="Pythonic" %}

```python
## ✅ Pythonic (Readable, follows PEP 8)
def is_threshold_reached(value):
    """Checks if the value exceeds the standard threshold."""
    return value > 10
```

{% endtab %}

{% endtabs %}

## Python Versions and Compatibility

### Python 2 vs Python 3

Python 3 was released in 2008 and is not backward-compatible
with Python 2. Python 2 reached end-of-life in 2020
and is no longer supported.

#### Key Differences

- **Print Statement:** `print "Hello"` (Python 2) vs `print("Hello")` (Python 3)
- **Integer Division:** `5/2 = 2` (Python 2) vs `5/2 = 2.5` (Python 3)
- **Unicode:** Strings are Unicode by default in Python 3
- **xrange:** `xrange()` (Python 2) vs `range()` (Python 3)

#### Current Python 3 Versions

- **3.12:** Latest stable release (October 2023)
- **3.11:** Performance-focused release with significant speed improvements
- **3.10:** Introduced pattern matching and improved error messages

### Choosing a Python Version

- Use the latest stable version for new projects
- Consider legacy dependencies when maintaining older codebases
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

## Summary

- Python is an interpreted language, making it flexible and easy to debug.
- Always use virtual environments to manage project dependencies.
- PEP 8 is the "Bible" of Python style—readability is a core feature
  of the language, not an afterthought.
- Python 3 is the current standard; Python 2 is no longer supported.
- Multiple implementations exist for different use cases and performance needs.
- The language's philosophy emphasizes simplicity, readability,
  and explicit design.

## Important Keywords

### **Bytecode**

Intermediate code (.pyc files) generated by Python compiler that runs on
the Python Virtual Machine (PVM). It's platform-independent and cached
for faster execution.

### **Dynamic Typing**

Variable types are determined at runtime rather than being declared in
advance. Allows flexibility but requires careful testing to avoid
type-related errors.

### **Garbage Collection**

Automatic memory management system that frees up memory occupied by
objects that are no longer referenced, preventing memory leaks.

### **REPL (Read-Eval-Print Loop)**

Interactive Python environment where you can type commands and get
immediate results. Essential for testing and learning.

### **PyPI (Python Package Index)**

Official repository for Python packages where developers can publish
and users can install third-party libraries.

### **Virtual Environment**

Isolated Python environment that contains specific Python version
and packages, preventing dependency conflicts between projects.

### **JIT Compilation (Just-In-Time)**

Compilation that happens during program execution rather than beforehand.
Used by PyPy to improve performance.

### **Type Hints**

Syntax for specifying expected data types (.pyi files) that improve
code documentation and enable static type checking.

### **PEP (Python Enhancement Proposal)**

Design documents that describe new features or community guidelines
for Python. PEP 8 defines style guidelines, PEP 20 defines the Zen of Python.

### **Multi-paradigm**

Support for multiple programming approaches: procedural (step-by-step),
object-oriented (objects and classes), and functional (pure functions).

### **Cross-platform**

Ability to run on multiple operating systems (Windows, macOS, Linux)
without code modification.

### **Standard Library**

Collection of modules included with Python installation, providing
"batteries included" functionality for common tasks.

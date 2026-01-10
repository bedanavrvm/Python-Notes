# File I/O and Context Managers

Interacting with the file system is a fundamental task for any programmer. In Python, this is handled through built-in functions and a powerful concept called a Context Manager, which ensures that system resources are managed safely and efficiently.

## File I/O: Reading and Writing

The basic function for interacting with files is `open()`. It requires a file path and a mode.

### Common File Modes

| Mode | Description |
|-------|-------------|
| `'r'` | Read (default). Fails if the file doesn't exist |
| `'w'` | Write. Creates a new file or overwrites an existing one |
| `'a'` | Append. Adds data to the end of an existing file |
| `'b'` | Binary. Used for non-text files (images, PDFs) |

### The "Old" Way (Manual Closing)

Before context managers, you had to manually close files. If an error occurred between open and close, the file might remain locked in memory, leading to resource leaks.

{% tabs %}

{% tab title="Manual close" %}

```python
f = open("data.txt", "w")
f.write("Hello World")

# If a crash happens here, the file stays open
f.close()
```

{% endtab %}

{% tab title="with statement" %}

```python
with open("data.txt", "r") as file:
    content = file.read()
    print(content)

## File is automatically closed here
```

{% endtab %}

{% endtabs %}

### The `with` Statement (Context Managers)

Python introduced the `with` statement to automate the setup and teardown of resources. This is the standard "Pythonic" way to handle files.

#### How it works

When a `with` block is entered, Python sets up a resource. When the block is exited (even if an error occurs), Python automatically calls a "cleanup" method (internally known as `__exit__`) to close the file.

### Modern Path Handling: `pathlib`

While older Python code uses the `os.path` module (which treats paths as strings), modern Python uses the `pathlib` module. `pathlib` treats paths as objects, making them much easier to manipulate across different operating systems (Windows uses `\`, while Linux/macOS use `/`).

#### Basic Path Operations

```python
from pathlib import Path

## Create a path object
current_dir = Path.cwd()
data_file = current_dir / "logs" / "test.txt"

## Check attributes
print(data_file.exists())  # True/False
print(data_file.suffix)  # .txt
print(data_file.stem)  # test

## Reading/Writing without explicit open()
data_file.write_text("Logging event...")
print(data_file.read_text())
```

#### Advanced Path Operations

```python
from pathlib import Path

# Directory operations
logs_dir = Path("logs")
logs_dir.mkdir(exist_ok=True)

# File operations
config_file = Path("config.json")
config_file.write_text('{"debug": true}')

# Path joining
data_path = Path("data") / "processed" / "output.csv"

# File globbing
for log_file in Path("logs").glob("*.log"):
    print(f"Processing {log_file}")

# File information
file_info = data_file.stat()
print(f"Size: {file_info.st_size} bytes")
print(f"Modified: {file_info.st_mtime}")
```

### File Encoding

When working with text files, encoding is crucial:

```python
# Specify encoding explicitly
with open("data.txt", "r", encoding="utf-8") as file:
    content = file.read()

# Binary files
with open("image.png", "rb") as file:
    image_data = file.read()

# Writing with encoding
with open("output.txt", "w", encoding="utf-8") as file:
    file.write("Hello, ")
```

## Custom Context Managers

You can create your own context managers to handle any resource that requires a "setup" and "teardown" phase (like a database connection or a network socket).

### 1. Class-based (`__enter__` and `__exit__`)

{% tabs %}

{% tab title="Class-based" %}

```python
class DatabaseConnection:
    def __enter__(self):
        print("Connecting to DB...")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Closing DB connection...")
        # Exception handling
        if exc_type:
            print(f"Error occurred: {exc_val}")
        return False  # Don't suppress exceptions

with DatabaseConnection() as db:
    print("Executing queries...")
```

{% endtab %}

{% tab title="Generator-based (contextlib)" %}

```python
from contextlib import contextmanager

@contextmanager
def temp_header():
    print("=== START ===")
    yield
    print("=== END ===")

with temp_header():
    print("I am the middle of a sandwich.")
```

{% endtab %}

{% endtabs %}

### 2. Generator-based (`contextlib`)

A more concise way to create context managers is using the `@contextmanager` decorator.

### 3. Multiple Resource Context Manager

```python
class FileProcessor:
    def __init__(self, input_file, output_file):
        self.input_file = input_file
        self.output_file = output_file
        self.input_handle = None
        self.output_handle = None

    def __enter__(self):
        self.input_handle = open(self.input_file, "r")
        self.output_handle = open(self.output_file, "w")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.input_handle:
            self.input_handle.close()
        if self.output_handle:
            self.output_handle.close()
        if exc_type:
            print(f"Error: {exc_val}")
        return False

# Usage
with FileProcessor("input.txt", "output.txt") as processor:
    # Both files are automatically managed
    content = processor.input_handle.read()
    processor.output_handle.write(f"Processed: {content}")
```

## Advanced File Operations

### File Locking

For concurrent access to files:

<details>
<summary>Show file locking example</summary>

```python
import fcntl
import os

def acquire_file_lock(file_path):
    """Acquire exclusive lock on file."""
    lock_file = f"{file_path}.lock"

    try:
        lock_fd = os.open(lock_file, os.O_CREAT | os.O_EXCL)
        fcntl.flock(lock_fd, fcntl.LOCK_EX)
        return True
    except OSError:
        return False

def with_file_lock(file_path, mode="r"):
    """Context manager for file locking."""
    if acquire_file_lock(file_path):
        try:
            with open(file_path, mode) as file:
                yield file
        finally:
            os.remove(f"{file_path}.lock")
    else:
        raise IOError(f"Could not acquire lock for {file_path}")

# Usage
with with_file_lock("shared_data.txt") as file:
    data = file.read()
    print(f"Read: {data}")
```

</details>

### Temporary Files

<details>
<summary>Show temporary files example</summary>

```python
import tempfile
import shutil

# Create temporary file
with tempfile.NamedTemporaryFile(mode='w', delete=False) as temp_file:
    temp_file.write("Temporary data")
    temp_path = temp_file.name

# File persists after context
with open(temp_path, 'r') as persistent_file:
    content = persistent_file.read()
    print(f"Temp file content: {content}")

# Temporary directory
with tempfile.TemporaryDirectory() as temp_dir:
    temp_file = temp_dir / "data.txt"
    temp_file.write_text("Temporary directory data")
    print(f"Temp dir: {temp_dir}")
# Directory automatically cleaned up
```

</details>

### File System Monitoring

<details>
<summary>Show file system monitoring example</summary>

```python
import os
import time
from pathlib import Path

def watch_directory(path, callback):
    """Simple file system watcher."""
    last_modified = {}

    for root, dirs, files in os.walk(path):
        for file in files:
            file_path = Path(root) / file
            current_modified = file_path.stat().st_mtime

            if file_path not in last_modified or current_modified > last_modified[file_path]:
                last_modified[file_path] = current_modified
                callback(file_path, "created" if file_path not in last_modified else "modified")

    print("Monitoring started...")

def file_changed_callback(file_path, event):
    print(f"File {file_path.name}: {event}")

# Usage
watch_directory("./data", file_changed_callback)
```

</details>

### Binary File Operations

<details>
<summary>Show binary file operations example</summary>

```python
import struct
import pickle

# Working with binary data
def write_binary_data(data, filename):
    """Write structured binary data."""
    with open(filename, "wb") as file:
        # Pack data
        packed_data = struct.pack('i', data)
        file.write(packed_data)

def read_binary_data(filename):
    """Read structured binary data."""
    with open(filename, "rb") as file:
        # Unpack data
        data = file.read(4)
        return struct.unpack('i', data)[0]

# Serialize objects
def save_object(obj, filename):
    with open(filename, "wb") as file:
        pickle.dump(obj, file)

def load_object(filename):
    with open(filename, "rb") as file:
        return pickle.load(file)
```

</details>

### Memory-Mapped Files

<details>
<summary>Show memory-mapped files example</summary>

```python
import mmap
import os

def memory_mapped_file(filename):
    """Create memory-mapped file for efficient access."""
    with open(filename, "r+b") as file:
        size = os.path.getsize(filename)
        return mmap.mmap(file.fileno(), 0, size)

# Usage
with memory_mapped_file("large_data.bin") as mmapped:
    # Access file as memory for fast operations
    data = mmapped.read(100)  # Read first 100 bytes
    mmapped.seek(50)       # Seek to position 50
    mmapped.write(b"Hello")  # Write at position 50
```

</details>

## Context Manager Best Practices

### Exception Handling

<details>
<summary>Show context manager exception-handling example</summary>

```python
class SafeFileWriter:
    def __init__(self, filename):
        self.filename = filename
        self.file = None

    def __enter__(self):
        try:
            self.file = open(self.filename, "w")
            return self.file
        except IOError as e:
            print(f"Failed to open {self.filename}: {e}")
            raise

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            try:
                self.file.close()
            except Exception as e:
                print(f"Error closing file: {e}")

        if exc_type:
            print(f"Exception in context: {exc_val}")
            # Don't suppress exceptions unless intended
            return False

# Usage
try:
    with SafeFileWriter("output.txt") as writer:
        writer.write("Important data")
except IOError as e:
    print(f"File operation failed: {e}")
```

</details>

### Resource Cleanup

<details>
<summary>Show cleanup-on-exit example</summary>

```python
import atexit
import tempfile
import shutil

def cleanup_on_exit():
    """Register cleanup function to run on program exit."""
    # Remove temporary files
    temp_dir = tempfile.gettempdir()
    if os.path.exists(temp_dir):
        shutil.rmtree(temp_dir)
    print("Temporary files cleaned up")

# Register cleanup
atexit.register(cleanup_on_exit)

# Your program will cleanup automatically on exit
```

</details>

### Performance Considerations

<details>
<summary>Show performance considerations example</summary>

```python
# Buffering for better performance
with open("large_file.txt", "r", buffering=8192) as file:
    # Larger buffer for better I/O performance
    data = file.read()  # Reads in larger chunks

# Line-by-line vs all-at-once
# For huge files, consider streaming
def process_large_file(filename):
    with open(filename, "r") as file:
        for line in file:  # Memory efficient
            process_line(line)

# Instead of
with open(filename, "r") as file:
    lines = file.readlines()  # Loads entire file into memory
    for line in lines:
        process_line(line)
```

</details>

## Summary

- Use `with` statements for automatic resource management
- `pathlib` provides modern, object-oriented file path handling
- Always specify encoding when working with text files
- Context managers prevent resource leaks through automatic cleanup
- Custom context managers can handle complex resource scenarios
- Consider performance implications for large file operations

## Important Keywords

### **File I/O**

Operations for reading from and writing to files, including text and binary data handling.

### **Context Manager**

Object that manages the setup and teardown of resources, ensuring proper cleanup through the `with` statement.

### **Resource Management**

Practice of properly acquiring, using, and releasing system resources like files, network connections, and memory.

### **File Handle**

Object representing an open file, providing methods and attributes for file operations.

### **File Mode**

String parameter specifying how a file should be opened (read, write, append, binary, etc.).

### **File Path**

Location of a file in the file system, can be manipulated using `pathlib` or `os.path` modules.

### **pathlib**

Modern Python module for object-oriented file system path manipulation, cross-platform compatible.

### **Encoding**

Process of converting between bytes and characters, crucial for proper text file handling across different languages.

### **Binary File**

File containing non-text data (images, executables, serialized objects) opened in binary mode.

### **File Locking**

Mechanism for controlling concurrent access to files, preventing data corruption in multi-process scenarios.

### **Temporary File**

File created for short-term storage, automatically cleaned up when no longer needed.

### **Resource Leak**

Situation where system resources are not properly released after use, leading to memory exhaustion or locked files.

### **Buffering**

Technique of optimizing I/O performance by reading/writing data in larger chunks rather than individual operations.

### **File System Monitoring**

Process of watching for changes in directories and files, enabling reactive programming patterns.

### **Serialization**

Process of converting complex objects to byte streams for storage or transmission, often using formats like pickle or JSON.

### **Memory Mapping**

Technique for mapping file contents directly into memory for efficient random access without loading entire file.

### **Atomic Operations**

File operations that complete entirely or not at all, preventing partial writes or corrupted data states.

### **File Permissions**

Access control mechanisms that determine which users or processes can read, write, or execute files.

### **Stream Processing**

Handling data as continuous flow rather than loading entire contents into memory, essential for large file processing.

### **Context Manager Protocol**

Interface consisting of `__enter__` and `__exit__` methods that objects must implement to work with `with` statements.

### **Exception Safety**

Design approach ensuring that exceptions in context managers are properly handled and don't suppress important errors unintentionally.

### **Cleanup Handler**

Process or method that ensures proper resource release, typically called automatically by context managers.

### **atexit Module**

Python module for registering functions to be called when the Python interpreter shuts down.

### **File Descriptor**

Mechanism for customizing attribute access on objects, enabling computed or validated properties.

### **Path Manipulation**

Operations for creating, joining, and transforming file system paths using object-oriented interfaces.

### **Cross-Platform Compatibility**

Writing code that works correctly across different operating systems (Windows, Linux, macOS) for file operations.

### **I/O Boundaries**

Limits and constraints imposed by the operating system on file operations, such as maximum file sizes or path lengths.

### **Concurrent Access**

Multiple processes or threads accessing the same file simultaneously, requiring coordination mechanisms.

### **Data Integrity**

Ensuring that file contents remain consistent and uncorrupted, often through checksums or atomic writes.

### **File Metadata**

Information about files such as creation time, modification time, size, and permissions, accessible through system calls or path objects.

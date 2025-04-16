# 6. Modules and Packages 📚

[<- Back to Functions](./05-functions.md) | [Next: Object-Oriented Programming ->](./07-oop.md)

## Table of Contents

- [Modules](#modules)
- [Importing](#importing)
- [Packages](#packages)
- [Standard Library](#standard-library)
- [Creating Modules](#creating-modules)
- [Creating Packages](#creating-packages)
- [Namespace Packages](#namespace-packages)
- [Distribution](#distribution)

## Modules

A module is a file containing Python definitions and statements intended for reuse. The file name is the module name with the suffix `.py` added.

### Why Use Modules?

- **Code organization**: Break code into logical units
- **Code reuse**: Write once, use multiple times
- **Namespace management**: Avoid naming conflicts

### Module Structure

```python
# example_module.py

"""This is the module docstring that describes the module purpose."""

# Module constants
PI = 3.14159
MAX_VALUE = 1000

# Module functions
def square(x):
    """Return the square of a number."""
    return x ** 2

def cube(x):
    """Return the cube of a number."""
    return x ** 3

# Module classes
class Circle:
    """A class representing a circle."""
    def __init__(self, radius):
        self.radius = radius
        
    def area(self):
        """Calculate the area of the circle."""
        return PI * self.radius ** 2
        
# Code that runs when the module is executed directly
if __name__ == "__main__":
    print("This module is being run directly.")
    test_radius = 5
    test_circle = Circle(test_radius)
    print(f"Area of circle with radius {test_radius}: {test_circle.area()}")
```

## Importing

Python provides various ways to import modules and their components.

### Basic Import Statements

```python
# Import the entire module
import math
print(math.pi)  # 3.141592653589793
print(math.sqrt(16))  # 4.0

# Import specific items from a module
from math import pi, sqrt
print(pi)  # 3.141592653589793
print(sqrt(16))  # 4.0

# Import all items from a module (not recommended due to namespace pollution)
from math import *
print(pi)  # 3.141592653589793
print(sqrt(16))  # 4.0

# Import with an alias
import math as m
print(m.pi)  # 3.141592653589793

# Import an item with an alias
from math import sqrt as square_root
print(square_root(16))  # 4.0
```

### Import Locations and Order

Python looks for modules in the following locations (in order):
1. Built-in modules
2. Modules in directories listed in `sys.path`:
   - Directory containing the script being run
   - PYTHONPATH environment variable
   - Installation-dependent default directories

```python
import sys
print(sys.path)  # List of module search paths
```

### Reloading Modules

Modules are imported only once per interpreter session. If the module changes, you need to reload it:

```python
import importlib
import my_module

# ... module changes ...

# Reload the module to get the changes
importlib.reload(my_module)
```

## Packages

A package is a way of organizing related modules in a directory hierarchy. Any directory containing an `__init__.py` file is considered a package.

### Basic Package Structure

```
my_package/
│
├── __init__.py        # Makes the directory a package
├── module1.py         # A module in the package
├── module2.py         # Another module
│
└── subpackage/        # A nested package
    ├── __init__.py    # Makes the directory a subpackage
    ├── module3.py     # A module in the subpackage
    └── module4.py     # Another module
```

### Importing from Packages

```python
# Import a module from a package
import my_package.module1
my_package.module1.some_function()

# Import a function from a module in a package
from my_package.module1 import some_function
some_function()

# Import a module with an alias
import my_package.module1 as m1
m1.some_function()

# Import a module from a subpackage
import my_package.subpackage.module3
my_package.subpackage.module3.another_function()

# Direct import from subpackage
from my_package.subpackage import module3
module3.another_function()
```

### `__init__.py` Files

The `__init__.py` file can be empty or can contain initialization code for the package:

```python
# my_package/__init__.py

"""My package provides useful utilities."""

# Package-level constants
VERSION = "1.0.0"

# Import specific modules or items to make them available at the package level
from .module1 import some_function
from .module2 import SomeClass

# Define what gets imported with 'from my_package import *'
__all__ = ['some_function', 'SomeClass', 'VERSION']
```

With this `__init__.py`, you can use:

```python
from my_package import some_function, SomeClass, VERSION
```

## Standard Library

Python comes with a rich standard library of modules for common tasks.

### Common Standard Library Modules

```python
# math - Mathematical functions
import math
print(math.pi)
print(math.sqrt(16))
print(math.floor(4.7))

# random - Random number generation
import random
print(random.random())  # Random float between 0 and 1
print(random.randint(1, 10))  # Random integer between 1 and 10
print(random.choice(["apple", "banana", "cherry"]))  # Random item from list

# datetime - Date and time manipulation
import datetime
now = datetime.datetime.now()
print(now)
print(now.year, now.month, now.day)
print(now + datetime.timedelta(days=10))  # 10 days from now

# os - Operating system interface
import os
print(os.getcwd())  # Current working directory
print(os.listdir())  # Files in current directory
os.mkdir("new_directory")  # Create a directory

# sys - System-specific parameters and functions
import sys
print(sys.platform)  # Operating system platform
print(sys.version)  # Python version
print(sys.argv)  # Command line arguments

# json - JSON encoding and decoding
import json
data = {"name": "Alice", "age": 30, "city": "New York"}
json_string = json.dumps(data)
print(json_string)
parsed_data = json.loads(json_string)
print(parsed_data["name"])
```

### Standard Library Categorization

The standard library includes modules for:
- Text processing
- Data types and algorithms
- Mathematical operations
- File and directory access
- Data persistence
- Data compression
- Networking and internet protocols
- Structured markup processing
- Internet data handling
- Multimedia services
- Internationalization
- Program frameworks
- Graphical user interfaces
- Development tools
- Runtime services
- Language tools

## Creating Modules

Creating your own modules is straightforward.

### Basic Module Creation

```python
# mymath.py
"""A module with math-related functions."""

def add(a, b):
    """Add two numbers."""
    return a + b

def subtract(a, b):
    """Subtract b from a."""
    return a - b

def multiply(a, b):
    """Multiply two numbers."""
    return a * b

def divide(a, b):
    """Divide a by b."""
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
```

### Using the Module

```python
# Using the mymath module
import mymath

print(mymath.add(5, 3))  # 8
print(mymath.multiply(4, 6))  # 24
```

### The `if __name__ == "__main__"` Idiom

This idiom allows code to be run when the module is executed directly but not when imported:

```python
# test_module.py
"""A module demonstrating the __name__ == "__main__" idiom."""

def main():
    """Main function that runs when the module is executed directly."""
    print("Running main function")
    print("This module was executed directly")

if __name__ == "__main__":
    main()
else:
    print("This module was imported")
```

When executed directly:
```
$ python test_module.py
Running main function
This module was executed directly
```

When imported:
```python
import test_module
# Output: This module was imported
```

## Creating Packages

Creating your own packages involves organizing modules into directories with `__init__.py` files.

### Basic Package Creation

1. Create a directory structure:
   ```
   mypackage/
   ├── __init__.py
   ├── module1.py
   └── module2.py
   ```

2. Create the `__init__.py` file:
   ```python
   """My package provides useful utilities."""
   
   # Package version
   __version__ = "0.1.0"
   ```

3. Create module files:
   ```python
   # module1.py
   """Module 1 in mypackage."""
   
   def function1():
       """Function in module1."""
       return "Function 1 result"
   ```
   
   ```python
   # module2.py
   """Module 2 in mypackage."""
   
   def function2():
       """Function in module2."""
       return "Function 2 result"
   ```

4. Use the package:
   ```python
   import mypackage.module1
   print(mypackage.module1.function1())
   
   from mypackage import module2
   print(module2.function2())
   ```

### Relative Imports

Within a package, you can use relative imports to reference other modules in the package:

```python
# mypackage/subpackage/module3.py

# Absolute import
from mypackage.module1 import function1

# Relative imports
from ..module1 import function1  # Import from parent package
from . import module4  # Import from same package
from .module4 import function4  # Import from module in same package
```

## Namespace Packages

Python 3.3+ introduced namespace packages, which don't require an `__init__.py` file. This allows package parts to be spread across multiple directories.

### Creating Namespace Packages

```
site-packages/
├── myproject/  # Part 1 of namespace package
│   └── module1.py
└── more_myproject/  # Part 2 of namespace package
    └── module2.py
```

The package name is `myproject` and it includes modules from both directories.

```python
import myproject.module1  # From the first directory
import myproject.module2  # From the second directory
```

## Distribution

To share your package with others, you can create a distribution package.

### Creating a Package for Distribution

1. Create a `setup.py` file at the root of your project:
   ```python
   from setuptools import setup, find_packages
   
   setup(
       name="mypackage",
       version="0.1.0",
       author="Your Name",
       author_email="your.email@example.com",
       description="A short description of the package",
       long_description=open("README.md").read(),
       long_description_content_type="text/markdown",
       url="https://github.com/yourusername/mypackage",
       packages=find_packages(),
       classifiers=[
           "Programming Language :: Python :: 3",
           "License :: OSI Approved :: MIT License",
           "Operating System :: OS Independent",
       ],
       python_requires=">=3.6",
   )
   ```

2. Create a package structure:
   ```
   mypackage_project/
   ├── LICENSE
   ├── README.md
   ├── setup.py
   └── mypackage/
       ├── __init__.py
       ├── module1.py
       └── module2.py
   ```

3. Build the package:
   ```bash
   pip install build
   python -m build
   ```
   This creates distribution files in the `dist/` directory.

4. Install the package locally:
   ```bash
   pip install -e .
   ```

5. Upload to PyPI:
   ```bash
   pip install twine
   twine upload dist/*
   ```

### Using a Published Package

Once your package is published on PyPI, others can install it with:

```bash
pip install mypackage
```

And use it in their Python code:

```python
import mypackage
from mypackage import module1

module1.function1()
```

---

[<- Back to Functions](./05-functions.md) | [Next: Object-Oriented Programming ->](./07-oop.md)
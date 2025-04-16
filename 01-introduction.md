# 1. Introduction to Python 🐍

[<- Back to Main Note](./README.md) | [Next: Basic Syntax and Data Types ->](./02-syntax-data-types.md)

## Table of Contents

- [What is Python?](#what-is-python)
- [Python Installation](#python-installation)
- [Python Interpreter](#python-interpreter)
- [Your First Python Program](#your-first-python-program)
- [Python Philosophy](#python-philosophy)

## What is Python?

Python is a high-level, interpreted programming language known for its readability and simplicity. Created by Guido van Rossum and first released in 1991, Python has become one of the most popular programming languages globally.

### Key characteristics of Python:

- **Easy to learn and read**: Clean syntax with significant whitespace
- **Interpreted**: No compilation step is required
- **Dynamically typed**: Variable types are determined at runtime
- **Garbage-collected**: Memory management is automatic
- **Multi-paradigm**: Supports procedural, object-oriented, and functional programming
- **Extensive standard library**: "Batteries included" philosophy
- **Large ecosystem**: Rich set of third-party packages

Python's versatility allows it to be used in various domains, including:
- Web development
- Data analysis and science
- Machine learning and AI
- Automation and scripting
- Scientific computing
- Game development

## Python Installation

### Installing Python on different operating systems:

#### Windows
```
1. Visit python.org/downloads and download the latest version
2. Run the installer (check "Add Python to PATH")
3. Verify installation: open Command Prompt and type 'python --version'
```

#### macOS
```
1. Install Homebrew (if not already installed): /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
2. Install Python: brew install python
3. Verify installation: python3 --version
```

#### Linux (Ubuntu/Debian)
```
1. Update package lists: sudo apt update
2. Install Python: sudo apt install python3 python3-pip
3. Verify installation: python3 --version
```

### Setting up a virtual environment

Virtual environments allow you to create isolated Python environments for projects:

```bash
# Create a virtual environment
python -m venv my_project_env

# Activate the environment
# Windows
my_project_env\\Scripts\\activate
# macOS/Linux
source my_project_env/bin/activate

# Deactivate when done
deactivate
```

## Python Interpreter

Python is an interpreted language, which means you can run Python code directly without compiling it first.

### Interactive Mode (REPL)

REPL stands for Read-Evaluate-Print Loop, providing an interactive way to execute Python code:

```bash
# Start the Python interpreter
python

# Now you're in interactive mode
>>> print("Hello, World!")
Hello, World!
>>> 2 + 3
5
>>> exit()  # or Ctrl+Z (Windows) or Ctrl+D (Unix) to exit
```

The interactive mode is excellent for:
- Testing small code snippets
- Learning Python syntax
- Debugging
- Quick calculations

## Your First Python Program

### Hello World

Create a file named `hello.py` with the following content:

```python
# This is a comment
print("Hello, World!")
```

Run it from the command line:

```bash
python hello.py
```

Output:
```
Hello, World!
```

### Basic Script Structure

```python
#!/usr/bin/env python3
"""
This is a docstring that describes the script.
It can span multiple lines.
"""

# Import statements come at the top
import math
import random

# Constants are typically in ALL_CAPS
PI = 3.14159
MAX_ATTEMPTS = 5

# Function definitions
def greet(name):
    """Return a greeting message."""
    return f"Hello, {name}!"

# Main execution block
if __name__ == "__main__":
    print(greet("Python Learner"))
    print(f"The value of PI is approximately {PI}")
    print(f"A random number between 1 and 10: {random.randint(1, 10)}")
```

## Python Philosophy

Python's design philosophy is summarized in "The Zen of Python", accessible by typing `import this` in the Python interpreter:

```
>>> import this
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

These principles guide Python's development and usage, encouraging clean, readable, and maintainable code.

---

[<- Back to Main Note](./README.md) | [Next: Basic Syntax and Data Types ->](./02-syntax-data-types.md)
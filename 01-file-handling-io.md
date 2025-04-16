# 1. File Handling and I/O 📂

[<- Back to Main Note](./README.md) | [Next: Error Handling ->](./02-error-handling.md)

## Table of Contents

- [Basic File Operations](#basic-file-operations)
- [Working with Paths](#working-with-paths)
- [Advanced File Operations](#advanced-file-operations)
- [Serialization](#serialization)
- [Working with CSV and Excel](#working-with-csv-and-excel)

## Basic File Operations

File handling in Python is straightforward with built-in functions.

### Opening and Closing Files

```python
# Basic file opening - always use with for proper resource management
with open('filename.txt', 'r') as file:
    content = file.read()
    # File automatically closes when block exits

# Common file modes
# 'r' - Read (default)
# 'w' - Write (creates new file or truncates existing)
# 'a' - Append
# 'b' - Binary mode
# 't' - Text mode (default)
# '+' - Update (read and write)
```

### Reading Files

```python
# Read entire file at once
with open('filename.txt', 'r') as file:
    content = file.read()  # Returns entire file as a string

# Read line by line
with open('filename.txt', 'r') as file:
    for line in file:  # Memory efficient for large files
        print(line, end='')

# Read all lines into a list
with open('filename.txt', 'r') as file:
    lines = file.readlines()  # Returns list of lines

# Read specific number of bytes
with open('filename.txt', 'r') as file:
    chunk = file.read(1024)  # Read 1024 bytes
```

### Writing Files

```python
# Write to a file (creates or truncates)
with open('output.txt', 'w') as file:
    file.write('Hello, World!\n')
    file.write('Another line.\n')

# Append to a file
with open('output.txt', 'a') as file:
    file.write('This gets added to the end.\n')

# Write multiple lines at once
lines = ['Line 1\n', 'Line 2\n', 'Line 3\n']
with open('output.txt', 'w') as file:
    file.writelines(lines)  # No newlines added automatically
```

## Working with Paths

Python provides robust path handling via the `pathlib` module (modern approach) or `os.path` (traditional approach).

### Pathlib Module (Python 3.4+)

```python
from pathlib import Path

# Create path objects
current_dir = Path.cwd()  # Current working directory
home_dir = Path.home()    # User's home directory
file_path = Path('folder/subfolder/file.txt')

# Path components
print(file_path.name)       # 'file.txt'
print(file_path.stem)       # 'file'
print(file_path.suffix)     # '.txt'
print(file_path.parent)     # Path('folder/subfolder')

# Joining paths (cross-platform)
new_path = current_dir / 'data' / 'output.txt'

# Check if exists
if file_path.exists():
    print(f"{file_path} exists")

# File operations
if file_path.is_file():
    print("It's a file")
if file_path.is_dir():
    print("It's a directory")

# Creating directories
new_dir = Path('new_directory')
new_dir.mkdir(exist_ok=True)         # Single directory
new_dir.mkdir(parents=True, exist_ok=True)  # Create parent dirs if needed

# Listing directory contents
for item in Path('.').iterdir():
    print(item)

# Finding files with pattern matching
for python_file in Path('.').glob('*.py'):
    print(python_file)

# Recursive search
for python_file in Path('.').rglob('*.py'):
    print(python_file)
```

### OS Path Module (Traditional)

```python
import os

# Current directory
current_dir = os.getcwd()

# Join paths (cross-platform)
file_path = os.path.join('folder', 'subfolder', 'file.txt')

# Path components
dirname = os.path.dirname(file_path)    # 'folder/subfolder'
basename = os.path.basename(file_path)  # 'file.txt'
filename, ext = os.path.splitext(basename)  # 'file', '.txt'

# Normalize path
normalized = os.path.normpath('/path//to/./file')

# Absolute path
abs_path = os.path.abspath('relative/path')

# Check if exists
if os.path.exists(file_path):
    print(f"{file_path} exists")

# Directory operations
if not os.path.exists('new_dir'):
    os.mkdir('new_dir')                 # Single directory
    os.makedirs('path/to/nested/dir')   # Recursive directory creation

# List directory contents
contents = os.listdir('.')
```

## Advanced File Operations

### File Seek and Tell

```python
with open('file.txt', 'r') as file:
    # Get current position
    position = file.tell()
    
    # Read 5 characters
    content = file.read(5)
    
    # Move to specific position (byte offset)
    file.seek(0)  # Back to start
    
    # Seek relative to current position
    file.seek(10, 1)  # 10 bytes forward from current position
    
    # Seek relative to end
    file.seek(-20, 2)  # 20 bytes before end of file
```

### Temporary Files

```python
import tempfile

# Temporary file that's automatically deleted
with tempfile.TemporaryFile() as temp:
    temp.write(b'Binary data')
    temp.seek(0)
    data = temp.read()

# Named temporary file
with tempfile.NamedTemporaryFile(delete=False) as temp:
    print(f"Created temporary file at {temp.name}")
    # File persists after closing

# Temporary directory
with tempfile.TemporaryDirectory() as temp_dir:
    print(f"Created temporary directory at {temp_dir}")
    # Directory and contents deleted after block
```

### File Metadata and Permissions

```python
import os
from pathlib import Path

file_path = 'example.txt'

# Get file stats
stats = os.stat(file_path)
print(f"Size: {stats.st_size} bytes")
print(f"Modified: {stats.st_mtime}")
print(f"Permissions: {stats.st_mode}")

# Change file permissions
os.chmod(file_path, 0o755)  # rwxr-xr-x

# With pathlib
path = Path(file_path)
stats = path.stat()
print(f"Size: {stats.st_size} bytes")

# Change owner (Unix-like systems only)
# os.chown(file_path, uid, gid)
```

## Serialization

Serialization converts Python objects to storable or transmittable formats.

### JSON Serialization

```python
import json

# Python dictionary
data = {
    'name': 'John',
    'age': 30,
    'languages': ['Python', 'JavaScript', 'Go'],
    'active': True,
    'details': {
        'country': 'USA',
        'job': 'Developer'
    }
}

# Serialize to JSON string
json_string = json.dumps(data, indent=4)
print(json_string)

# Write JSON to file
with open('data.json', 'w') as file:
    json.dump(data, file, indent=4)

# Read JSON from file
with open('data.json', 'r') as file:
    loaded_data = json.load(file)

# Parse JSON string
parsed_data = json.loads(json_string)

# Custom object serialization
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

# Custom encoder function
def encode_person(obj):
    if isinstance(obj, Person):
        return {'name': obj.name, 'age': obj.age}
    raise TypeError(f"Object of type {type(obj)} is not JSON serializable")

person = Person("Alice", 25)
person_json = json.dumps(person, default=encode_person)
```

### Pickle (Python Object Serialization)

```python
import pickle

# Python objects
data = {
    'complex_object': [1, 2, 3, {'key': 'value'}, (4, 5, 6)],
    'class_instance': object()  # Can serialize almost any Python object
}

# Serialize to binary file
with open('data.pkl', 'wb') as file:
    pickle.dump(data, file)

# Deserialize from binary file
with open('data.pkl', 'rb') as file:
    loaded_data = pickle.load(file)

# To string and back
serialized = pickle.dumps(data)
deserialized = pickle.loads(serialized)

# WARNING: Pickle is not secure - never unpickle data from untrusted sources
```

### YAML Serialization

```python
import yaml  # Requires pyyaml package

# Python dictionary
data = {
    'name': 'John',
    'age': 30,
    'languages': ['Python', 'JavaScript', 'Go'],
    'details': {
        'country': 'USA',
        'job': 'Developer'
    }
}

# Convert Python object to YAML
yaml_string = yaml.dump(data, default_flow_style=False)
print(yaml_string)

# Write YAML to file
with open('data.yaml', 'w') as file:
    yaml.dump(data, file, default_flow_style=False)

# Read YAML from file
with open('data.yaml', 'r') as file:
    loaded_data = yaml.safe_load(file)  # safe_load prevents code execution

# Parse YAML string
parsed_data = yaml.safe_load(yaml_string)
```

## Working with CSV and Excel

### CSV Files

```python
import csv

# Writing CSV
data = [
    ['Name', 'Age', 'City'],
    ['John', 30, 'New York'],
    ['Alice', 25, 'Chicago'],
    ['Bob', 35, 'Boston']
]

with open('people.csv', 'w', newline='') as file:
    writer = csv.writer(file)
    writer.writerows(data)  # Write all rows at once
    
    # Or write row by row
    # for row in data:
    #     writer.writerow(row)

# Reading CSV
with open('people.csv', 'r', newline='') as file:
    reader = csv.reader(file)
    for row in reader:
        print(row)

# Using DictReader and DictWriter
dict_data = [
    {'Name': 'John', 'Age': 30, 'City': 'New York'},
    {'Name': 'Alice', 'Age': 25, 'City': 'Chicago'},
    {'Name': 'Bob', 'Age': 35, 'City': 'Boston'}
]

# Write CSV with headers
with open('people_dict.csv', 'w', newline='') as file:
    fieldnames = ['Name', 'Age', 'City']
    writer = csv.DictWriter(file, fieldnames=fieldnames)
    
    writer.writeheader()
    writer.writerows(dict_data)

# Read CSV as dictionaries
with open('people_dict.csv', 'r', newline='') as file:
    reader = csv.DictReader(file)
    for row in reader:
        print(f"{row['Name']} is {row['Age']} years old from {row['City']}")
```

### Excel Files

```python
# Requires openpyxl package for modern Excel files
import openpyxl

# Create a new workbook
workbook = openpyxl.Workbook()
sheet = workbook.active
sheet.title = "People"

# Add headers
headers = ['Name', 'Age', 'City']
for col_num, header in enumerate(headers, 1):
    sheet.cell(row=1, column=col_num, value=header)

# Add data
data = [
    ['John', 30, 'New York'],
    ['Alice', 25, 'Chicago'],
    ['Bob', 35, 'Boston']
]

for row_num, row_data in enumerate(data, 2):
    for col_num, value in enumerate(row_data, 1):
        sheet.cell(row=row_num, column=col_num, value=value)

# Save the workbook
workbook.save('people.xlsx')

# Read an Excel file
read_workbook = openpyxl.load_workbook('people.xlsx')
read_sheet = read_workbook.active

# Iterate through rows
for row in read_sheet.iter_rows(min_row=2, values_only=True):
    name, age, city = row
    print(f"{name} is {age} years old from {city}")
```

---

[<- Back to Main Note](./README.md) | [Next: Error Handling ->](./02-error-handling.md)

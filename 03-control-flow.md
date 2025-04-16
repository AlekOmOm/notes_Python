# 3. Control Flow ⚓

[<- Back to Syntax and Data Types](./02-syntax-data-types.md) | [Next: Data Structures ->](./04-data-structures.md)

## Table of Contents

- [Conditional Statements](#conditional-statements)
- [Loops](#loops)
- [Control Flow Tools](#control-flow-tools)
- [Exception Handling Basics](#exception-handling-basics)
- [Context Managers](#context-managers)

## Conditional Statements

Conditional statements allow your program to make decisions based on the evaluation of conditions.

### if, elif, else

The basic structure of conditional statements in Python:

```python
# Simple if statement
age = 18
if age >= 18:
    print("You are an adult")

# if-else statement
temperature = 15
if temperature > 25:
    print("It's warm outside")
else:
    print("It's cool outside")

# if-elif-else chain
score = 85
if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
elif score >= 60:
    grade = "D"
else:
    grade = "F"
print(f"Your grade is {grade}")
```

### Conditional Expressions (Ternary Operator)

Python provides a concise way to write simple if-else statements:

```python
# Standard if-else
if age >= 18:
    status = "adult"
else:
    status = "minor"

# Equivalent conditional expression (ternary operator)
status = "adult" if age >= 18 else "minor"
```

### Truthy and Falsy Values

In Python, values can be evaluated in a boolean context:

- **Falsy values**: `False`, `None`, `0`, `0.0`, `''` (empty string), `[]` (empty list), `()` (empty tuple), `{}` (empty dict), `set()` (empty set)
- **Truthy values**: Everything else

```python
# Using truthy and falsy values
name = ""
if name:  # Checks if name is not empty
    print(f"Hello, {name}")
else:
    print("Name is empty")

items = []
if not items:  # Checks if items is empty
    print("The list is empty")
```

## Loops

Loops allow you to execute a block of code repeatedly.

### for Loops

The `for` loop in Python iterates over the items of any sequence (like a list, tuple, string) or other iterable objects:

```python
# Iterating over a list
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)

# Iterating over a string
for char in "Python":
    print(char)

# Iterating over a range
for i in range(5):  # 0, 1, 2, 3, 4
    print(i)

# Iterating with index using enumerate
for index, value in enumerate(fruits):
    print(f"{index}: {value}")

# Iterating over multiple sequences using zip
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]
for name, age in zip(names, ages):
    print(f"{name} is {age} years old")
```

### while Loops

The `while` loop executes as long as a condition is true:

```python
# Basic while loop
count = 0
while count < 5:
    print(count)
    count += 1

# while loop with condition at the end
answer = ""
while True:
    answer = input("Continue? (yes/no): ")
    if answer.lower() in ["no", "n"]:
        break
```

### Loop Control Statements

Python provides statements to control the flow of loops:

```python
# break: exits the loop completely
for i in range(10):
    if i == 5:
        break  # Exit the loop when i equals 5
    print(i)  # Prints 0, 1, 2, 3, 4

# continue: skips the current iteration
for i in range(10):
    if i % 2 == 0:
        continue  # Skip even numbers
    print(i)  # Prints 1, 3, 5, 7, 9

# else clause in loops: executed when loop completes normally (not through break)
for i in range(5):
    print(i)
else:
    print("Loop completed normally")

# else clause is not executed if loop is exited with break
for i in range(5):
    if i == 3:
        break
    print(i)
else:
    print("This won't be printed")
```

## Control Flow Tools

### Comprehensions as Control Flow

List, dictionary, and set comprehensions provide a concise way to create collections with conditional logic:

```python
# List comprehension with condition
even_numbers = [x for x in range(10) if x % 2 == 0]  # [0, 2, 4, 6, 8]

# Conditional expression in comprehension
numbers = [x if x % 2 == 0 else -x for x in range(10)]  # [0, -1, 2, -3, 4, -5, 6, -7, 8, -9]

# Dictionary comprehension with condition
even_squares = {x: x**2 for x in range(10) if x % 2 == 0}  # {0: 0, 2: 4, 4: 16, 6: 36, 8: 64}
```

### all(), any()

The `all()` and `any()` functions provide a way to check if all or any items in an iterable are true:

```python
# all(): returns True if all elements are True
all([True, True, True])  # True
all([True, False, True])  # False
all(x > 0 for x in [1, 2, 3])  # True
all(x > 0 for x in [1, -2, 3])  # False

# any(): returns True if at least one element is True
any([False, False, True])  # True
any([False, False, False])  # False
any(x > 5 for x in [1, 2, 3])  # False
any(x > 5 for x in [1, 7, 3])  # True
```

### match Statement (Python 3.10+)

The `match` statement (introduced in Python 3.10) provides pattern matching capabilities:

```python
# Basic match statement
status = 404
match status:
    case 400:
        print("Bad request")
    case 404:
        print("Not found")
    case 418:
        print("I'm a teapot")
    case _:  # Default case
        print("Something's wrong with the internet")

# Pattern matching with data structures
point = (3, 4)
match point:
    case (0, 0):
        print("Origin")
    case (0, y):
        print(f"Y-axis, y={y}")
    case (x, 0):
        print(f"X-axis, x={x}")
    case (x, y):
        print(f"Point at ({x}, {y})")
    case _:
        print("Not a point")
```

## Exception Handling Basics

Exception handling allows your program to respond to runtime errors gracefully.

### try, except, else, finally

The basic structure of exception handling in Python:

```python
# Basic try-except
try:
    x = 10 / 0  # Raises ZeroDivisionError
except ZeroDivisionError:
    print("Cannot divide by zero!")

# Handling multiple exceptions
try:
    value = int(input("Enter a number: "))
    result = 10 / value
except ValueError:
    print("Invalid input: not a number")
except ZeroDivisionError:
    print("Cannot divide by zero")

# Capturing the exception object
try:
    with open("nonexistent_file.txt", "r") as f:
        content = f.read()
except FileNotFoundError as error:
    print(f"Error: {error}")

# else clause: executed if no exception occurs
try:
    value = int(input("Enter a positive number: "))
    if value <= 0:
        raise ValueError("Value must be positive")
except ValueError as error:
    print(f"Error: {error}")
else:
    print(f"You entered {value}")

# finally clause: always executed
try:
    f = open("example.txt", "r")
    content = f.read()
except FileNotFoundError:
    print("File not found")
finally:
    # This ensures the file is closed whether an exception occurred or not
    if 'f' in locals() and not f.closed:
        f.close()
    print("File operation complete")
```

## Context Managers

Context managers (using the `with` statement) provide a clean way to manage resources:

```python
# File handling with context manager
with open("example.txt", "w") as f:
    f.write("Hello, World!")
# File is automatically closed after the block

# Multiple context managers
with open("input.txt", "r") as in_file, open("output.txt", "w") as out_file:
    content = in_file.read()
    out_file.write(content.upper())

# Custom context managers using contextlib
from contextlib import contextmanager

@contextmanager
def managed_resource():
    print("Acquiring resource")
    resource = {"data": "valuable data"}
    try:
        yield resource
    finally:
        print("Releasing resource")

with managed_resource() as resource:
    print(f"Using resource: {resource['data']}")
```

Context managers are particularly useful for:
- File operations
- Database connections
- Network connections
- Locks and semaphores
- Temporarily changing settings

---

[<- Back to Syntax and Data Types](./02-syntax-data-types.md) | [Next: Data Structures ->](./04-data-structures.md)
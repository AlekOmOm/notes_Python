# 2. Basic Syntax and Data Types 📦

[<- Back to Introduction](./01-introduction.md) | [Next: Control Flow ->](./03-control-flow.md)

## Table of Contents

- [Syntax Fundamentals](#syntax-fundamentals)
- [Variables and Assignment](#variables-and-assignment)
- [Basic Data Types](#basic-data-types)
- [Type Conversion](#type-conversion)
- [Operators](#operators)
- [Input and Output](#input-and-output)

## Syntax Fundamentals

Python's syntax is designed to be readable and straightforward, with some key characteristics:

### Indentation

Python uses indentation to define code blocks, not braces or keywords:

```python
# Correct indentation
if x > 5:
    print("x is greater than 5")
    if x > 10:
        print("x is also greater than 10")

# Incorrect indentation would cause an IndentationError
```

### Comments

```python
# This is a single-line comment

"""
This is a multi-line comment or docstring
It can span multiple lines
"""
```

### Line Continuation

Long lines can be broken using:

```python
# Backslash for line continuation
long_string = "This is a very long string that \
spans multiple lines in the code"

# Implicit continuation inside parentheses, brackets, or braces
coordinates = (
    1.0, 2.0,
    3.0, 4.0
)

# Trailing commas are allowed
names = [
    "Alice",
    "Bob",
    "Charlie",
]
```

## Variables and Assignment

### Variable Naming Rules

- Must start with a letter or underscore
- Can contain letters, numbers, and underscores
- Case-sensitive
- Cannot use Python keywords

```python
# Valid variable names
name = "John"
age_in_years = 30
_private_variable = "hidden"
totalCount = 100  # Valid but not PEP 8 compliant

# Invalid variable names
# 2names = "Invalid"  # Cannot start with a number
# class = "Student"   # Cannot use Python keywords
```

### Multiple Assignment

```python
# Assigning the same value to multiple variables
x = y = z = 0

# Unpacking a collection
a, b, c = 1, 2, 3
coordinates = [4, 5, 6]
x, y, z = coordinates
```

## Basic Data Types

Python has several built-in data types:

### Numeric Types

```python
# Integer
x = 5
big_num = 1_000_000  # Underscores for readability

# Float
pi = 3.14159
scientific = 1.23e4  # 12300.0

# Complex
complex_num = 3 + 4j
```

### Strings

```python
# String literals
single_quotes = 'Hello'
double_quotes = "World"
triple_quotes = '''Multiple
lines'''

# String operations
greeting = "Hello"
name = "Alice"
full_greeting = greeting + " " + name  # Concatenation
repeated = greeting * 3  # "HelloHelloHello"

# String indexing and slicing
first_char = greeting[0]  # 'H'
substring = greeting[1:4]  # 'ell'

# String methods
uppercase = greeting.upper()  # "HELLO"
greeting_length = len(greeting)  # 5
```

### Boolean

```python
is_active = True
is_complete = False

# Expressions that evaluate to boolean
is_valid = 5 > 3  # True
```

### None Type

```python
value = None  # Represents absence of a value
```

## Type Conversion

Python allows conversion between data types:

```python
# Explicit type conversion
integer_value = 5
float_value = float(integer_value)  # 5.0
string_value = str(integer_value)   # "5"
boolean_value = bool(integer_value) # True

# Common type conversions
int_from_float = int(3.9)     # 3 (truncates, doesn't round)
int_from_string = int("42")   # 42
float_from_string = float("3.14")  # 3.14
```

## Operators

### Arithmetic Operators

```python
a, b = 10, 3

addition = a + b        # 13
subtraction = a - b     # 7
multiplication = a * b  # 30
division = a / b        # 3.3333... (always returns float)
floor_division = a // b # 3 (integer division, discards remainder)
modulus = a % b         # 1 (remainder of division)
exponentiation = a ** b # 1000 (10 to the power of 3)
```

### Assignment Operators

```python
x = 5        # Basic assignment
x += 3       # Same as x = x + 3
x -= 2       # Same as x = x - 2
x *= 4       # Same as x = x * 4
x /= 2       # Same as x = x / 2
x //= 2      # Same as x = x // 2
x %= 3       # Same as x = x % 3
x **= 2      # Same as x = x ** 2
```

### Comparison Operators

```python
a, b = 5, 10

equal = a == b           # False
not_equal = a != b       # True
greater_than = a > b     # False
less_than = a < b        # True
greater_equal = a >= b   # False
less_equal = a <= b      # True
```

### Logical Operators

```python
x, y = True, False

and_result = x and y  # False (both must be True)
or_result = x or y    # True (at least one must be True)
not_result = not x    # False (negates the boolean value)
```

### Identity Operators

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

identical_object = a is c    # True (a and c refer to the same object)
different_object = a is b    # False (a and b are equivalent but separate objects)
not_identical = a is not b   # True
```

### Membership Operators

```python
list_values = [1, 2, 3, 4, 5]

in_list = 3 in list_values        # True
not_in_list = 10 not in list_values  # True
```

## Input and Output

### Basic Output

```python
print("Hello, World!")  # Simple string output
print("Value:", 42)     # Multiple items
print("x =", 5, "y =", 10)  # Multiple values

# Formatting with f-strings (Python 3.6+)
name = "Alice"
age = 30
print(f"{name} is {age} years old.")

# Formatting with str.format()
print("{} is {} years old.".format(name, age))
print("{name} is {age} years old.".format(name="Bob", age=25))
```

### Basic Input

```python
# Getting input from user
name = input("Enter your name: ")
print(f"Hello, {name}!")

# Converting input to a specific type
age = int(input("Enter your age: "))
height = float(input("Enter your height in meters: "))
```

### Formatted String Literals (f-strings)

```python
name = "Charlie"
items = 3
price = 19.99

# Basic formatting
print(f"{name} bought {items} items.")

# Number formatting
print(f"Total cost: ${items * price:.2f}")  # Two decimal places

# Alignment and padding
print(f"{name:15} | {items:>5} | ${price:<8.2f}")

# Date formatting
import datetime
today = datetime.datetime.now()
print(f"Today is {today:%B %d, %Y}")
```

---

[<- Back to Introduction](./01-introduction.md) | [Next: Control Flow ->](./03-control-flow.md)
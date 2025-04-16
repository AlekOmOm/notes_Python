# 5. Functions ⚙️

[<- Back to Data Structures](./04-data-structures.md) | [Next: Modules and Packages ->](./06-modules-packages.md)

## Table of Contents

- [Function Basics](#function-basics)
- [Parameters and Arguments](#parameters-and-arguments)
- [Return Values](#return-values)
- [Variable Scope](#variable-scope)
- [Lambda Functions](#lambda-functions)
- [Closures](#closures)
- [Decorators](#decorators)
- [Recursion](#recursion)
- [Function as First-Class Objects](#functions-as-first-class-objects)
- [Functional Programming Tools](#functional-programming-tools)

## Function Basics

Functions are reusable blocks of code that perform a specific task. They help organize code, avoid repetition, and improve readability.

### Defining and Calling Functions

```python
# Basic function definition
def greet():
    """This function prints a greeting."""
    print("Hello, World!")

# Calling the function
greet()  # Output: Hello, World!

# Function with parameters
def greet_person(name):
    """Greet a specific person."""
    print(f"Hello, {name}!")

greet_person("Alice")  # Output: Hello, Alice!
```

### Docstrings

Docstrings document what a function does. They appear right after the function definition and are enclosed in triple quotes:

```python
def calculate_area(radius):
    """
    Calculate the area of a circle.
    
    Parameters:
        radius (float): The radius of the circle
        
    Returns:
        float: The area of the circle
    """
    return 3.14159 * radius ** 2

# Accessing the docstring
help(calculate_area)
print(calculate_area.__doc__)
```

## Parameters and Arguments

Functions can accept input values (parameters) and be called with specific values (arguments).

### Positional Arguments

```python
def describe_pet(animal_type, pet_name):
    """Display information about a pet."""
    print(f"I have a {animal_type} named {pet_name}.")

# Calling with positional arguments (order matters)
describe_pet("hamster", "Harry")  # I have a hamster named Harry.
```

### Keyword Arguments

```python
# Calling with keyword arguments (order doesn't matter)
describe_pet(pet_name="Harry", animal_type="hamster")  # I have a hamster named Harry.
```

### Default Parameter Values

```python
def describe_pet(pet_name, animal_type="dog"):
    """Display information about a pet."""
    print(f"I have a {animal_type} named {pet_name}.")

# Using the default value
describe_pet("Willie")  # I have a dog named Willie.
# Overriding the default value
describe_pet("Harry", "hamster")  # I have a hamster named Harry.
```

### Variable-Length Arguments

```python
# *args: Variable number of positional arguments (tuple)
def make_pizza(*toppings):
    """Print the list of toppings that have been requested."""
    print("Making a pizza with the following toppings:")
    for topping in toppings:
        print(f"- {topping}")

make_pizza("pepperoni")
make_pizza("mushrooms", "green peppers", "extra cheese")

# **kwargs: Variable number of keyword arguments (dictionary)
def build_profile(first, last, **user_info):
    """Build a dictionary containing everything we know about a user."""
    profile = {"first_name": first, "last_name": last}
    for key, value in user_info.items():
        profile[key] = value
    return profile

user = build_profile("Albert", "Einstein",
                     location="Princeton",
                     field="Physics",
                     awards=["Nobel Prize"])
print(user)
```

### Unpacking Arguments

```python
# Unpacking a list into positional arguments
def point(x, y, z):
    print(f"Point coordinates: ({x}, {y}, {z})")

coordinates = [3, 7, 9]
point(*coordinates)  # Point coordinates: (3, 7, 9)

# Unpacking a dictionary into keyword arguments
person = {"name": "Alice", "age": 30, "job": "Engineer"}
def describe_person(name, age, job):
    print(f"{name} is {age} years old and works as an {job}.")

describe_person(**person)  # Alice is 30 years old and works as an Engineer.
```

## Return Values

Functions can return values to the caller using the `return` statement.

### Returning a Single Value

```python
def square(number):
    """Return the square of a number."""
    return number ** 2

result = square(5)
print(result)  # 25
```

### Returning Multiple Values

```python
def get_dimensions():
    """Return width and height."""
    return 1200, 800

# Return value is a tuple that can be unpacked
width, height = get_dimensions()
print(f"Width: {width}, Height: {height}")
```

### Early Returns and Conditional Returns

```python
def absolute_value(number):
    """Return the absolute value of a number."""
    if number >= 0:
        return number
    else:
        return -number

# Alternative implementation with early return
def absolute_value_alt(number):
    """Return the absolute value of a number."""
    if number >= 0:
        return number
    return -number
```

## Variable Scope

The scope of a variable determines where it's accessible from.

### Local and Global Scope

```python
# Global variable
message = "Global message"

def print_message():
    # Local variable
    local_message = "Local message"
    print(message)  # Can access global variable
    print(local_message)  # Can access local variable

print_message()
print(message)  # Global variable is still accessible
# print(local_message)  # Error: local_message is not defined
```

### Modifying Global Variables

```python
counter = 0

def increment_counter():
    # Must declare global to modify the global variable
    global counter
    counter += 1
    print(f"Counter: {counter}")

increment_counter()  # Counter: 1
increment_counter()  # Counter: 2
```

### Nonlocal Variables

```python
def outer_function():
    outer_var = "outer"
    
    def inner_function():
        # Must declare nonlocal to modify the outer function's variable
        nonlocal outer_var
        outer_var = "modified"
        print(f"Inner: {outer_var}")
    
    inner_function()
    print(f"Outer: {outer_var}")

outer_function()
# Output:
# Inner: modified
# Outer: modified
```

## Lambda Functions

Lambda functions are small, anonymous functions defined with the `lambda` keyword.

### Basic Lambda Syntax

```python
# Regular function
def add(x, y):
    return x + y

# Equivalent lambda function
add_lambda = lambda x, y: x + y

print(add(5, 3))       # 8
print(add_lambda(5, 3))  # 8
```

### Lambda Functions with Built-in Functions

```python
# Using lambda with sorted()
students = [
    {"name": "Alice", "grade": 85},
    {"name": "Bob", "grade": 92},
    {"name": "Charlie", "grade": 78}
]

# Sort by grade
sorted_by_grade = sorted(students, key=lambda student: student["grade"])

# Using lambda with map()
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))  # [1, 4, 9, 16, 25]

# Using lambda with filter()
even_numbers = list(filter(lambda x: x % 2 == 0, numbers))  # [2, 4]
```

## Closures

A closure is a function that remembers the values in the enclosing scope even if they are not present in memory anymore.

### Creating Closures

```python
def make_multiplier(factor):
    """Create a function that multiplies its argument by factor."""
    def multiplier(x):
        return x * factor
    return multiplier

# Create multiplier functions
double = make_multiplier(2)
triple = make_multiplier(3)

print(double(5))  # 10
print(triple(5))  # 15
```

### Practical Use of Closures

```python
def counter():
    count = 0
    
    def increment():
        nonlocal count
        count += 1
        return count
    
    return increment

# Create counter
my_counter = counter()
print(my_counter())  # 1
print(my_counter())  # 2
print(my_counter())  # 3

# Create another counter (independent state)
another_counter = counter()
print(another_counter())  # 1
```

## Decorators

Decorators are functions that modify the behavior of other functions.

### Basic Decorator

```python
def decorator_function(original_function):
    def wrapper_function(*args, **kwargs):
        print(f"Wrapper executed before {original_function.__name__}")
        result = original_function(*args, **kwargs)
        print(f"Wrapper executed after {original_function.__name__}")
        return result
    return wrapper_function

@decorator_function
def display():
    print("Display function ran")

display()
# Output:
# Wrapper executed before display
# Display function ran
# Wrapper executed after display
```

### Decorators with Arguments

```python
def repeat(n=1):
    def decorator(func):
        def wrapper(*args, **kwargs):
            result = None
            for _ in range(n):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(n=3)
def greet(name):
    print(f"Hello {name}")

greet("World")
# Output:
# Hello World
# Hello World
# Hello World
```

## Recursion

Recursion is a technique where a function calls itself to solve a problem.

### Basic Recursion

```python
def factorial(n):
    """Calculate the factorial of n using recursion."""
    # Base case
    if n == 0 or n == 1:
        return 1
    # Recursive case
    else:
        return n * factorial(n - 1)

print(factorial(5))  # 120 (5 * 4 * 3 * 2 * 1)
```

### Fibonacci Sequence with Recursion

```python
def fibonacci(n):
    """Return the nth number in the Fibonacci sequence."""
    if n <= 0:
        return 0
    elif n == 1:
        return 1
    else:
        return fibonacci(n - 1) + fibonacci(n - 2)

for i in range(10):
    print(fibonacci(i), end=" ")  # 0 1 1 2 3 5 8 13 21 34
```

## Functions as First-Class Objects

In Python, functions are first-class objects, meaning they can be passed around and used as arguments.

### Passing Functions as Arguments

```python
def apply_function(func, value):
    """Apply a function to a value and return the result."""
    return func(value)

def square(x):
    return x ** 2

def cube(x):
    return x ** 3

print(apply_function(square, 5))  # 25
print(apply_function(cube, 5))    # 125
```

### Returning Functions

```python
def get_math_function(operation):
    """Return a math function based on the operation."""
    def add(x, y):
        return x + y
    
    def subtract(x, y):
        return x - y
    
    def multiply(x, y):
        return x * y
    
    def divide(x, y):
        return x / y
    
    if operation == "add":
        return add
    elif operation == "subtract":
        return subtract
    elif operation == "multiply":
        return multiply
    elif operation == "divide":
        return divide
    else:
        return None

# Get a function and call it
math_func = get_math_function("multiply")
print(math_func(5, 3))  # 15
```

## Functional Programming Tools

Python provides several tools for functional programming.

### map()

```python
# Apply a function to each item in an iterable
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))
print(squared)  # [1, 4, 9, 16, 25]

# Using a regular function with map
def celsius_to_fahrenheit(celsius):
    return (celsius * 9/5) + 32

temperatures_c = [0, 10, 20, 30, 40]
temperatures_f = list(map(celsius_to_fahrenheit, temperatures_c))
print(temperatures_f)  # [32.0, 50.0, 68.0, 86.0, 104.0]
```

### filter()

```python
# Filter elements based on a function that returns True/False
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
even = list(filter(lambda x: x % 2 == 0, numbers))
print(even)  # [2, 4, 6, 8, 10]

# Using a regular function with filter
def is_positive(num):
    return num > 0

values = [-2, -1, 0, 1, 2]
positive_values = list(filter(is_positive, values))
print(positive_values)  # [1, 2]
```

### reduce()

```python
from functools import reduce

# Apply a function of two arguments cumulatively to the items of an iterable
numbers = [1, 2, 3, 4, 5]
product = reduce(lambda x, y: x * y, numbers)
print(product)  # 120 (1 * 2 * 3 * 4 * 5)

# Using a regular function with reduce
def add(x, y):
    return x + y

sum_result = reduce(add, numbers)
print(sum_result)  # 15 (1 + 2 + 3 + 4 + 5)
```

### partial()

```python
from functools import partial

def power(base, exponent):
    return base ** exponent

# Create new functions with partial application
square = partial(power, exponent=2)
cube = partial(power, exponent=3)

print(square(5))  # 25
print(cube(5))    # 125
```

---

[<- Back to Data Structures](./04-data-structures.md) | [Next: Modules and Packages ->](./06-modules-packages.md)
# 10. Advanced Python Concepts 🚀

[<- Back to Error Handling](./09-error-handling.md) | [Next: Concurrency and Parallelism ->](./11-concurrency.md)

## Table of Contents

- [Decorators](#decorators)
- [Generators and Iterators](#generators-and-iterators)
- [Context Managers](#context-managers)
- [Metaclasses](#metaclasses)
- [Descriptors](#descriptors)
- [Function Annotations and Type Hints](#function-annotations-and-type-hints)
- [Functional Programming](#functional-programming)
- [Memory Management and Optimization](#memory-management-and-optimization)

## Decorators

Decorators are a powerful way to modify or enhance functions or methods without changing their implementation.

### Function Decorators

```python
# Basic decorator pattern
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Something is happening before the function is called.")
        result = func(*args, **kwargs)
        print("Something is happening after the function is called.")
        return result
    return wrapper

# Applying a decorator
@my_decorator
def say_hello(name):
    print(f"Hello, {name}!")

# Equivalent to:
# say_hello = my_decorator(say_hello)

say_hello("Alice")
# Output:
# Something is happening before the function is called.
# Hello, Alice!
# Something is happening after the function is called.
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
    print(f"Hello, {name}!")

greet("Bob")  # Will print "Hello, Bob!" three times
```

### Class Decorators

```python
def singleton(cls):
    instances = {}
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return get_instance

@singleton
class Config:
    def __init__(self):
        self.settings = {}
        
    def set(self, key, value):
        self.settings[key] = value

# Both variables reference the same instance
config1 = Config()
config2 = Config()
print(config1 is config2)  # True
```

### Preserving Metadata

The `functools.wraps` decorator helps preserve the original function's metadata:

```python
import functools

def my_decorator(func):
    @functools.wraps(func)  # Preserves __name__, __doc__, etc.
    def wrapper(*args, **kwargs):
        print("Before function call")
        result = func(*args, **kwargs)
        print("After function call")
        return result
    return wrapper

@my_decorator
def add(a, b):
    """Add two numbers and return the result."""
    return a + b

print(add.__name__)  # "add" (without @wraps, it would be "wrapper")
print(add.__doc__)   # "Add two numbers and return the result."
```

### Common Built-in Decorators

```python
# @property - Creates a property from a method
class Circle:
    def __init__(self, radius):
        self._radius = radius
        
    @property
    def radius(self):
        return self._radius
        
    @radius.setter
    def radius(self, value):
        if value <= 0:
            raise ValueError("Radius must be positive")
        self._radius = value
        
    @property
    def area(self):
        return 3.14159 * self._radius ** 2

# @classmethod - Method that operates on the class, not instances
class MyClass:
    count = 0
    
    def __init__(self):
        MyClass.count += 1
        
    @classmethod
    def get_count(cls):
        return cls.count

# @staticmethod - Method that doesn't operate on instance or class
class MathUtils:
    @staticmethod
    def add(a, b):
        return a + b
```

## Generators and Iterators

### Iterators

An iterator is an object that implements the iterator protocol, which consists of the `__iter__()` and `__next__()` methods:

```python
class CountDown:
    def __init__(self, start):
        self.start = start
        
    def __iter__(self):
        # Return the iterator object (self)
        return self
        
    def __next__(self):
        if self.start <= 0:
            # Signal the end of iteration
            raise StopIteration
        self.start -= 1
        return self.start + 1

# Using the iterator
for num in CountDown(5):
    print(num)  # 5, 4, 3, 2, 1
```

### Generators

Generators are a simpler way to create iterators using functions with the `yield` statement:

```python
def countdown(start):
    while start > 0:
        yield start
        start -= 1

# Using the generator
for num in countdown(5):
    print(num)  # 5, 4, 3, 2, 1

# Generator expressions (similar to list comprehensions but create generators)
squares_gen = (x**2 for x in range(10))
print(next(squares_gen))  # 0
print(next(squares_gen))  # 1
```

### Generator Functions with Multiple Yields

```python
def fibonacci(n):
    a, b = 0, 1
    count = 0
    while count < n:
        yield a
        a, b = b, a + b
        count += 1

# Generate first 10 Fibonacci numbers
fib_nums = list(fibonacci(10))  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

### Generator Delegation with yield from

```python
def gen1():
    yield 1
    yield 2
    yield 3

def gen2():
    yield 4
    yield 5
    yield 6

def combined():
    yield from gen1()
    yield from gen2()

# Using the combined generator
for num in combined():
    print(num)  # 1, 2, 3, 4, 5, 6
```

### Generator State and send()

Generators can receive values back with the `send()` method:

```python
def echo_generator():
    while True:
        received = yield
        yield received * 2

gen = echo_generator()
next(gen)  # Prime the generator
print(gen.send(3))  # 6
next(gen)  # Advance to the next yield
print(gen.send(5))  # 10
```

## Context Managers

Context managers are used with the `with` statement to manage resources properly.

### Using with Statement

```python
# Built-in context manager for file handling
with open("example.txt", "w") as file:
    file.write("Hello, World!")
# File is automatically closed when the block exits
```

### Creating Context Managers with Classes

```python
class DatabaseConnection:
    def __init__(self, connection_string):
        self.connection_string = connection_string
        
    def __enter__(self):
        print(f"Connecting to {self.connection_string}")
        # In a real implementation, this would return a connection object
        self.connection = {"connected": True}
        return self.connection
        
    def __exit__(self, exc_type, exc_value, traceback):
        print("Closing connection")
        self.connection = None
        # Return False to propagate exceptions or True to suppress them
        return False

# Using the context manager
with DatabaseConnection("mysql://localhost:3306/mydb") as conn:
    print("Doing something with the connection")
    print(f"Connection status: {conn}")
```

### Creating Context Managers with contextlib

```python
import contextlib

@contextlib.contextmanager
def temporary_setting(settings, key, value):
    # Setup
    old_value = settings.get(key)
    settings[key] = value
    try:
        # The yield statement is where the with block executes
        yield
    finally:
        # Cleanup
        if old_value is None:
            del settings[key]
        else:
            settings[key] = old_value

# Using the context manager
settings = {"debug": False, "color": "blue"}
with temporary_setting(settings, "debug", True):
    print(f"Inside context: {settings}")  # {'debug': True, 'color': 'blue'}
print(f"Outside context: {settings}")  # {'debug': False, 'color': 'blue'}
```

### Nested Context Managers

```python
with open("input.txt", "r") as in_file, open("output.txt", "w") as out_file:
    content = in_file.read()
    out_file.write(content.upper())
```

## Metaclasses

Metaclasses are the "classes of classes" - they define how classes behave.

### Understanding Metaclasses

```python
# By default, classes are instances of type
class MyClass:
    pass
    
print(type(MyClass))  # <class 'type'>

# Creating a class dynamically with type
DynamicClass = type("DynamicClass", (object,), {"x": 10, "get_x": lambda self: self.x})
obj = DynamicClass()
print(obj.get_x())  # 10
```

### Custom Metaclass

```python
class Meta(type):
    def __new__(mcs, name, bases, attrs):
        # Add a new attribute to the class
        attrs["added_by_meta"] = True
        # Convert all method names to uppercase
        uppercase_attrs = {
            key.upper() if not key.startswith("__") else key: value
            for key, value in attrs.items()
        }
        return super().__new__(mcs, name, bases, uppercase_attrs)
        
    def __init__(cls, name, bases, attrs):
        print(f"Initialized class: {name}")
        super().__init__(name, bases, attrs)

# Using the metaclass
class MyClass(metaclass=Meta):
    def method(self):
        return "Hello"

# The method is now uppercase
obj = MyClass()
print(obj.METHOD())  # "Hello"
print(obj.added_by_meta)  # True
```

### Singleton Metaclass

```python
class Singleton(type):
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Logger(metaclass=Singleton):
    def __init__(self):
        self.logs = []
        
    def log(self, message):
        self.logs.append(message)

# Both logger instances are the same object
logger1 = Logger()
logger2 = Logger()
logger1.log("Test message")
print(logger2.logs)  # ["Test message"]
print(logger1 is logger2)  # True
```

## Descriptors

Descriptors are objects that define how attribute access is handled.

### Property Descriptor

```python
class Property:
    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        self.__doc__ = doc
        
    def __get__(self, instance, owner):
        if instance is None:
            return self
        if self.fget is None:
            raise AttributeError("unreadable attribute")
        return self.fget(instance)
        
    def __set__(self, instance, value):
        if self.fset is None:
            raise AttributeError("can't set attribute")
        self.fset(instance, value)
        
    def __delete__(self, instance):
        if self.fdel is None:
            raise AttributeError("can't delete attribute")
        self.fdel(instance)
```

### Custom Descriptors

```python
class Validation:
    def __init__(self, name=None):
        self.name = name
        
    def __set__(self, instance, value):
        instance.__dict__[self.name] = value
        
    def __delete__(self, instance):
        del instance.__dict__[self.name]

class Integer(Validation):
    def __set__(self, instance, value):
        if not isinstance(value, int):
            raise TypeError("Expected an integer")
        super().__set__(instance, value)

class String(Validation):
    def __set__(self, instance, value):
        if not isinstance(value, str):
            raise TypeError("Expected a string")
        super().__set__(instance, value)

class Person:
    name = String(name="name")
    age = Integer(name="age")
    
    def __init__(self, name, age):
        self.name = name
        self.age = age

# Using descriptors
person = Person("Alice", 30)
# person.age = "thirty"  # Raises TypeError: Expected an integer
```

## Function Annotations and Type Hints

Python 3.5+ introduced type hints to provide optional static typing.

### Basic Type Annotations

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"

def calculate_statistics(numbers: list[int]) -> tuple[float, float, float]:
    """Calculate the mean, median, and standard deviation."""
    n = len(numbers)
    mean = sum(numbers) / n
    sorted_numbers = sorted(numbers)
    median = sorted_numbers[n // 2] if n % 2 == 1 else (sorted_numbers[n // 2 - 1] + sorted_numbers[n // 2]) / 2
    variance = sum((x - mean) ** 2 for x in numbers) / n
    std_dev = variance ** 0.5
    return mean, median, std_dev
```

### Complex Type Hints

```python
from typing import Union, Optional, Callable, Dict, List, Tuple, Any

# Union type (can be one of several types)
def process_data(data: Union[str, bytes]) -> str:
    if isinstance(data, bytes):
        return data.decode('utf-8')
    return data

# Optional type (value or None)
def get_user(user_id: int, cache: Optional[Dict[int, str]] = None) -> str:
    if cache is not None and user_id in cache:
        return cache[user_id]
    return f"User {user_id}"

# Callable type (functions or callable objects)
def execute_function(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)

# Type aliases
Vector = List[float]
Matrix = List[Vector]

def dot_product(v1: Vector, v2: Vector) -> float:
    return sum(x * y for x, y in zip(v1, v2))

def matrix_multiply(m1: Matrix, m2: Matrix) -> Matrix:
    # Implementation details omitted
    return [[0.0]]
```

### Type Checking with mypy

```python
# example.py
def add(a: int, b: int) -> int:
    return a + b

result = add("3", "5")  # Type error: str instead of int
```

Run mypy to check types:
```bash
$ mypy example.py
example.py:4: error: Argument 1 to "add" has incompatible type "str"; expected "int"
example.py:4: error: Argument 2 to "add" has incompatible type "str"; expected "int"
```

### Type Annotations in Classes

```python
class BankAccount:
    def __init__(self, owner: str, balance: float = 0.0) -> None:
        self.owner: str = owner
        self.balance: float = balance
        
    def deposit(self, amount: float) -> float:
        self.balance += amount
        return self.balance
        
    def withdraw(self, amount: float) -> float:
        if amount > self.balance:
            raise ValueError("Insufficient funds")
        self.balance -= amount
        return self.balance
```

## Functional Programming

Python supports many functional programming paradigms.

### Pure Functions

Pure functions always produce the same output for the same input and have no side effects:

```python
# Pure function
def add(a, b):
    return a + b

# Impure function (has side effects)
def add_to_list(value, my_list=[]):  # Mutable default argument!
    my_list.append(value)
    return my_list
```

### Higher-order Functions

Functions that take other functions as arguments or return functions:

```python
def apply_operation(func, a, b):
    return func(a, b)

def add(a, b):
    return a + b

def multiply(a, b):
    return a * b

print(apply_operation(add, 5, 3))       # 8
print(apply_operation(multiply, 5, 3))  # 15
```

### Functional Tools

```python
from functools import reduce

# map: Apply function to every item in an iterable
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))  # [1, 4, 9, 16, 25]

# filter: Filter items by a function that returns True/False
even_numbers = list(filter(lambda x: x % 2 == 0, numbers))  # [2, 4]

# reduce: Apply a function to reduce a sequence to a single value
sum_result = reduce(lambda x, y: x + y, numbers)  # 15 (1+2+3+4+5)
```

### Function Composition

```python
def compose(*functions):
    """Compose multiple functions."""
    def composed_function(arg):
        result = arg
        for func in reversed(functions):
            result = func(result)
        return result
    return composed_function

def double(x):
    return x * 2

def increment(x):
    return x + 1

double_then_increment = compose(increment, double)
print(double_then_increment(5))  # 11 (5*2 + 1)
```

### Immutability

```python
# Using immutable data structures
original_tuple = (1, 2, 3)
new_tuple = original_tuple + (4, 5)  # Creates a new tuple
print(original_tuple)  # (1, 2, 3) - unchanged
print(new_tuple)       # (1, 2, 3, 4, 5)

# Using frozen sets (immutable sets)
immutable_set = frozenset([1, 2, 3])
# immutable_set.add(4)  # AttributeError: 'frozenset' object has no attribute 'add'
```

## Memory Management and Optimization

Understanding Python's memory management can help write more efficient code.

### Memory Management Basics

```python
import sys

# Check object size in bytes
x = 1
print(sys.getsizeof(x))  # Size of integer object

# Reference counting
import ctypes

def ref_count(obj):
    """Get the reference count of an object."""
    return ctypes.c_long.from_address(id(obj)).value

y = [1, 2, 3]
print(ref_count(y))  # Initial reference count
z = y  # Create another reference
print(ref_count(y))  # Reference count increased
```

### Memory Profiling

```python
# Using the memory_profiler package
# pip install memory_profiler

@profile
def memory_intensive_function():
    big_list = [0] * 1000000
    return sum(big_list)

# Run with: python -m memory_profiler script.py
```

### Optimizing Memory Usage

```python
# Using generators instead of lists for large sequences
def first_n_squares(n):
    # Memory-intensive approach
    # squares = [i**2 for i in range(n)]
    # return squares
    
    # Memory-efficient approach
    for i in range(n):
        yield i**2

# Using slots to reduce memory consumption
class Point:
    __slots__ = ['x', 'y']  # Restricts attributes and saves memory
    
    def __init__(self, x, y):
        self.x = x
        self.y = y

# Regular class
class PointRegular:
    def __init__(self, x, y):
        self.x = x
        self.y = y

# Compare memory usage
p1 = Point(1, 2)
p2 = PointRegular(1, 2)
print(sys.getsizeof(p1))  # Smaller
print(sys.getsizeof(p2))  # Larger
```

### Performance Optimization

```python
# Using built-in functions and libraries
import time

# Slow approach
def slow_sum(n):
    total = 0
    for i in range(n):
        total += i
    return total

# Fast approach (built-in function)
def fast_sum(n):
    return sum(range(n))

# Even faster approach (mathematical formula)
def fastest_sum(n):
    return (n * (n - 1)) // 2

# Timing comparison
start = time.time()
slow_sum(1000000)
print(f"Slow: {time.time() - start}")

start = time.time()
fast_sum(1000000)
print(f"Fast: {time.time() - start}")

start = time.time()
fastest_sum(1000000)
print(f"Fastest: {time.time() - start}")
```

---

[<- Back to Error Handling](./09-error-handling.md) | [Next: Concurrency and Parallelism ->](./11-concurrency.md)
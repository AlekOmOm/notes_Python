# 7. Object-Oriented Programming 🧩

[<- Back to Modules and Packages](./06-modules-packages.md) | [Next: File Handling and I/O ->](./08-file-handling.md)

## Table of Contents

- [Introduction to OOP](#introduction-to-oop)
- [Classes and Objects](#classes-and-objects)
- [Attributes and Methods](#attributes-and-methods)
- [Constructors and Destructors](#constructors-and-destructors)
- [Inheritance](#inheritance)
- [Polymorphism](#polymorphism)
- [Encapsulation](#encapsulation)
- [Abstraction](#abstraction)
- [Method Types](#method-types)
- [Properties](#properties)
- [Magic Methods](#magic-methods)
- [Advanced OOP Concepts](#advanced-oop-concepts)

## Introduction to OOP

Object-Oriented Programming (OOP) is a programming paradigm based on the concept of "objects," which can contain data (attributes) and code (methods). Python is a multi-paradigm language that supports OOP principles.

Key concepts in OOP:
- **Classes**: Blueprints for creating objects
- **Objects**: Instances of classes
- **Attributes**: Data associated with objects
- **Methods**: Functions that operate on objects
- **Inheritance**: Creating new classes from existing ones
- **Polymorphism**: Objects can take different forms
- **Encapsulation**: Hiding implementation details
- **Abstraction**: Simplifying complex systems

## Classes and Objects

### Defining a Class

```python
class Dog:
    # Class attribute (shared by all instances)
    species = "Canis familiaris"
    
    # Instance method
    def __init__(self, name, age):
        # Instance attributes (unique to each instance)
        self.name = name
        self.age = age
    
    # Another instance method
    def speak(self, sound):
        return f"{self.name} says {sound}"
```

### Creating Objects (Instances)

```python
# Creating instances of the Dog class
buddy = Dog("Buddy", 9)
miles = Dog("Miles", 4)

# Accessing attributes
print(buddy.name)  # "Buddy"
print(miles.age)   # 4
print(buddy.species)  # "Canis familiaris" (access class attribute)

# Calling methods
print(buddy.speak("Woof"))  # "Buddy says Woof"
print(miles.speak("Bow Wow"))  # "Miles says Bow Wow"
```

## Attributes and Methods

### Instance vs. Class Attributes

```python
class Circle:
    # Class attribute
    pi = 3.14159
    
    def __init__(self, radius):
        # Instance attribute
        self.radius = radius
    
    def area(self):
        return Circle.pi * self.radius ** 2  # Access class attribute
    
    def circumference(self):
        return 2 * self.pi * self.radius  # Can also access via self
```

### Adding and Modifying Attributes

```python
# Create a Circle
c1 = Circle(5)

# Modify an existing attribute
c1.radius = 7

# Add a new attribute to an instance
c1.color = "blue"  # Note: other Circle instances won't have this attribute

# Add a new attribute to a class
Circle.unit = "cm"  # All instances will now have this attribute
```

## Constructors and Destructors

### Constructor (`__init__`)

The `__init__` method is called when an object is created:

```python
class Person:
    def __init__(self, name, age):
        print(f"Initializing {name}")
        self.name = name
        self.age = age
```

### Destructor (`__del__`)

The `__del__` method is called when an object is about to be destroyed:

```python
class Resource:
    def __init__(self, name):
        self.name = name
        print(f"Resource {name} has been allocated")
        
    def __del__(self):
        print(f"Resource {self.name} is being freed")
```

## Inheritance

Inheritance allows a class to inherit attributes and methods from another class:

### Basic Inheritance

```python
# Base class (parent)
class Animal:
    def __init__(self, name, species):
        self.name = name
        self.species = species
        
    def make_sound(self):
        print("Some generic sound")
        
    def __str__(self):
        return f"{self.name} is a {self.species}"

# Derived class (child)
class Cat(Animal):
    def __init__(self, name, breed):
        # Call the parent class's __init__
        super().__init__(name, species="Cat")
        self.breed = breed
        
    def make_sound(self):
        # Override the parent's method
        print("Meow!")
```

### Multiple Inheritance

Python supports multiple inheritance, where a class can inherit from multiple parent classes:

```python
class Swimmer:
    def swim(self):
        return "Swimming"

class Flyer:
    def fly(self):
        return "Flying"

class Duck(Swimmer, Flyer):
    def __init__(self, name):
        self.name = name
```

### Method Resolution Order (MRO)

Method Resolution Order defines the order in which Python looks for methods in the inheritance hierarchy:

```python
class A:
    def method(self):
        return "A method"

class B(A):
    pass

class C(A):
    def method(self):
        return "C method"

class D(B, C):
    pass

d = D()
print(d.method())  # "C method" (follows MRO)
print(D.__mro__)   # Shows the MRO: (<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>)
```

## Polymorphism

Polymorphism allows objects of different classes to be treated as objects of a common superclass:

```python
class Shape:
    def area(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
        
    def area(self):
        return self.width * self.height

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
        
    def area(self):
        return 3.14 * self.radius ** 2

# Polymorphic function
def print_area(shape):
    print(f"Area: {shape.area()}")

# Using different shapes
rectangle = Rectangle(5, 10)
circle = Circle(7)

print_area(rectangle)  # Area: 50
print_area(circle)     # Area: 153.86
```

## Encapsulation

Encapsulation is the bundling of data and methods that work on that data within a single unit (class) and restricting access to some of the object's components:

### Private Attributes and Methods

Python doesn't have true private variables, but uses a naming convention of single leading underscore for protected and double for private:

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner  # public
        self._balance = balance  # protected (convention)
        self.__transactions = []  # private (name mangling)
        
    def deposit(self, amount):
        if amount > 0:
            self._balance += amount
            self.__transactions.append(f"Deposit: {amount}")
            return True
        return False
    
    def withdraw(self, amount):
        if 0 < amount <= self._balance:
            self._balance -= amount
            self.__transactions.append(f"Withdrawal: {amount}")
            return True
        return False
    
    def get_balance(self):
        return self._balance
    
    def get_transactions(self):
        # Return a copy to prevent modification
        return self.__transactions.copy()
```

## Abstraction

Abstraction is the concept of hiding the complex implementation details and showing only the necessary features:

### Abstract Base Classes

Python's `abc` module provides abstract base classes:

```python
from abc import ABC, abstractmethod

class Vehicle(ABC):
    @abstractmethod
    def start_engine(self):
        pass
    
    @abstractmethod
    def stop_engine(self):
        pass
    
    def horn(self):
        return "Honk!"

class Car(Vehicle):
    def start_engine(self):
        return "Car engine started"
    
    def stop_engine(self):
        return "Car engine stopped"

# This would raise an error
# v = Vehicle()  # TypeError: Can't instantiate abstract class

# This works because Car implements the abstract methods
c = Car()
print(c.start_engine())  # "Car engine started"
print(c.horn())          # "Honk!"
```

## Method Types

### Instance Methods

Regular methods that operate on instance attributes and have access to `self`:

```python
class MyClass:
    def instance_method(self):
        return f"Called instance_method of {self}"
```

### Class Methods

Methods that operate on class attributes and have access to `cls`:

```python
class MyClass:
    count = 0
    
    def __init__(self):
        MyClass.count += 1
    
    @classmethod
    def get_count(cls):
        return cls.count
    
    @classmethod
    def create_with_name(cls, name):
        instance = cls()
        instance.name = name
        return instance
```

### Static Methods

Methods that don't operate on instance or class attributes:

```python
class Math:
    @staticmethod
    def add(x, y):
        return x + y
    
    @staticmethod
    def is_prime(n):
        if n <= 1:
            return False
        for i in range(2, int(n ** 0.5) + 1):
            if n % i == 0:
                return False
        return True
```

## Properties

Properties allow controlled access to attributes:

```python
class Person:
    def __init__(self, name, age):
        self._name = name
        self._age = age
    
    @property
    def name(self):
        """Get the person's name."""
        return self._name
    
    @name.setter
    def name(self, value):
        """Set the person's name."""
        if not isinstance(value, str):
            raise TypeError("Name must be a string")
        self._name = value
    
    @property
    def age(self):
        """Get the person's age."""
        return self._age
    
    @age.setter
    def age(self, value):
        """Set the person's age."""
        if not isinstance(value, int):
            raise TypeError("Age must be an integer")
        if value < 0:
            raise ValueError("Age cannot be negative")
        self._age = value

# Using properties
person = Person("Alice", 30)
print(person.name)  # "Alice" (calls the getter)
person.name = "Bob"  # Calls the setter
# person.age = -5  # Raises ValueError
# person.age = "thirty"  # Raises TypeError
```

## Magic Methods

Magic methods (also called dunder methods for "double underscore") customize how objects of a class behave in various Python operations:

### String Representation

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __str__(self):
        """String representation for end users."""
        return f"Point({self.x}, {self.y})"
    
    def __repr__(self):
        """String representation for developers."""
        return f"Point(x={self.x}, y={self.y})"
```

### Operator Overloading

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __add__(self, other):
        """Vector addition: v1 + v2"""
        return Vector(self.x + other.x, self.y + other.y)
    
    def __sub__(self, other):
        """Vector subtraction: v1 - v2"""
        return Vector(self.x - other.x, self.y - other.y)
    
    def __mul__(self, scalar):
        """Scalar multiplication: v * scalar"""
        return Vector(self.x * scalar, self.y * scalar)
    
    def __rmul__(self, scalar):
        """Scalar multiplication (right): scalar * v"""
        return self.__mul__(scalar)
    
    def __eq__(self, other):
        """Vector equality: v1 == v2"""
        if not isinstance(other, Vector):
            return False
        return self.x == other.x and self.y == other.y
    
    def __str__(self):
        return f"Vector({self.x}, {self.y})"
```

### Container Methods

```python
class Deck:
    def __init__(self):
        self.cards = ["A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"]
    
    def __len__(self):
        """Length of the deck: len(deck)"""
        return len(self.cards)
    
    def __getitem__(self, position):
        """Access an item: deck[position]"""
        return self.cards[position]
    
    def __setitem__(self, position, value):
        """Set an item: deck[position] = value"""
        self.cards[position] = value
    
    def __delitem__(self, position):
        """Delete an item: del deck[position]"""
        del self.cards[position]
    
    def __contains__(self, card):
        """Check if card is in deck: card in deck"""
        return card in self.cards
    
    def __iter__(self):
        """Iterator for the deck: for card in deck"""
        return iter(self.cards)
```

### Context Manager Methods

```python
class DatabaseConnection:
    def __init__(self, connection_string):
        self.connection_string = connection_string
        self.connection = None
    
    def __enter__(self):
        """Called when entering a with block"""
        print(f"Connecting to {self.connection_string}")
        self.connection = {"status": "connected"}
        return self.connection
    
    def __exit__(self, exc_type, exc_value, traceback):
        """Called when exiting a with block"""
        print("Closing database connection")
        self.connection = None
        # Return False to propagate exceptions or True to suppress them
        return False
```

## Advanced OOP Concepts

### Mixins

Mixins are classes designed to be "mixed in" with other classes to add functionality:

```python
class LoggerMixin:
    def log(self, message):
        print(f"[LOG] {message}")
    
    def warn(self, message):
        print(f"[WARNING] {message}")
    
    def error(self, message):
        print(f"[ERROR] {message}")

class Database(LoggerMixin):
    def connect(self):
        self.log("Database connected")
    
    def disconnect(self):
        self.log("Database disconnected")
```

### Descriptors

Descriptors provide a powerful way to customize attribute access:

```python
class Validation:
    def __init__(self, name=None, **kwargs):
        self.name = name
        for key, value in kwargs.items():
            setattr(self, key, value)
    
    def __set__(self, instance, value):
        instance.__dict__[self.name] = value
    
    def __delete__(self, instance):
        del instance.__dict__[self.name]

class Typed(Validation):
    expected_type = type(None)
    
    def __set__(self, instance, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(f"Expected {self.expected_type}")
        super().__set__(instance, value)

class Integer(Typed):
    expected_type = int

class String(Typed):
    expected_type = str

class Person:
    name = String(name="name")
    age = Integer(name="age")
    
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

### Metaclasses

Metaclasses are classes that define the behavior of other classes:

```python
class SingletonMeta(type):
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(SingletonMeta, cls).__call__(*args, **kwargs)
        return cls._instances[cls]

class Singleton(metaclass=SingletonMeta):
    def __init__(self, value):
        self.value = value

# Usage
s1 = Singleton(1)
s2 = Singleton(2)
print(s1.value)  # 1
print(s2.value)  # 1 (same instance)
print(s1 is s2)  # True
```

### Class Decorators

Class decorators modify the behavior of a class:

```python
def singleton(cls):
    """Decorator that makes a class a Singleton."""
    instances = {}
    
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    
    return get_instance

@singleton
class Configuration:
    def __init__(self, settings=None):
        self.settings = settings or {}
```

---

[<- Back to Modules and Packages](./06-modules-packages.md) | [Next: File Handling and I/O ->](./08-file-handling.md)
# 4. Data Structures 🏗️

[<- Back to Control Flow](./03-control-flow.md) | [Next: Functions ->](./05-functions.md)

## Table of Contents

- [Introduction to Data Structures](#introduction-to-data-structures)
- [Overview of Python's Built-in Data Structures](#overview-of-pythons-built-in-data-structures)
  - [Lists](#lists)
  - [Tuples](#tuples)
  - [Sets](#sets)
  - [Dictionaries](#dictionaries)
- [Collections Module](#collections-module)
- [Choosing the Right Data Structure](#choosing-the-right-data-structure)
- [Comprehensions](#comprehensions)
- [Deep Dive Sub-topics](#deep-dive-sub-topics)

## Introduction to Data Structures

Data structures provide organized ways to store and manipulate data. Selecting the right data structure for a specific task can significantly impact code efficiency, readability, and performance. Python offers several built-in data structures that cater to different needs.

Key considerations when choosing data structures:
- Type of data to be stored
- Access patterns (random access, sequential access)
- Memory constraints
- Required operations (search, insertion, deletion)
- Time complexity requirements

## Overview of Python's Built-in Data Structures

### Lists

Lists are ordered, mutable sequences that can store elements of different types.

```python
# Creating a list
fruits = ["apple", "banana", "cherry"]

# Adding elements
fruits.append("orange")  # ["apple", "banana", "cherry", "orange"]
fruits.insert(1, "mango")  # ["apple", "mango", "banana", "cherry", "orange"]

# Removing elements
fruits.remove("banana")  # ["apple", "mango", "cherry", "orange"]
last_fruit = fruits.pop()  # "orange", fruits = ["apple", "mango", "cherry"]
```

**Key characteristics:**
- Ordered (elements have a defined order)
- Mutable (can be modified after creation)
- Allow duplicates
- Indexed by integers starting from 0
- Can contain elements of different data types

[Detailed List Guide](./04a-lists.md)

### Tuples

Tuples are ordered, immutable sequences.

```python
# Creating a tuple
coordinates = (10, 20)
person = ("John", 30, "Developer")

# Accessing elements
x, y = coordinates  # Unpacking
name = person[0]

# Cannot modify a tuple
# person[1] = 31  # TypeError: 'tuple' object does not support item assignment

# But we can create a new tuple based on an existing one
person = person[:1] + (31,) + person[2:]  # ("John", 31, "Developer")
```

**Key characteristics:**
- Ordered
- Immutable (cannot be modified after creation)
- Allow duplicates
- Can contain elements of different data types
- Often used for fixed data that shouldn't change

### Sets

Sets are unordered collections of unique elements.

```python
# Creating a set
unique_colors = {"red", "green", "blue"}
numbers = set([1, 2, 2, 3, 4, 4])  # {1, 2, 3, 4}

# Adding and removing elements
unique_colors.add("yellow")
unique_colors.remove("green")  # Raises KeyError if not present
unique_colors.discard("black")  # No error if not present

# Set operations
set1 = {1, 2, 3, 4}
set2 = {3, 4, 5, 6}
union = set1 | set2  # {1, 2, 3, 4, 5, 6}
intersection = set1 & set2  # {3, 4}
difference = set1 - set2  # {1, 2}
symmetric_difference = set1 ^ set2  # {1, 2, 5, 6}
```

**Key characteristics:**
- Unordered (no defined order)
- Mutable
- No duplicates (automatically removes duplicates)
- Elements must be immutable (cannot contain lists or dictionaries)
- Efficient for membership testing, eliminating duplicates

### Dictionaries

Dictionaries store key-value pairs, providing efficient lookup by key.

```python
# Creating a dictionary
person = {
    "name": "Alice",
    "age": 30,
    "skills": ["Python", "JavaScript", "SQL"]
}

# Accessing values
name = person["name"]  # "Alice"
age = person.get("age")  # 30
experience = person.get("experience", 0)  # Default value if key doesn't exist

# Modifying a dictionary
person["email"] = "alice@example.com"  # Add new key-value pair
person["age"] = 31  # Update existing value
del person["skills"]  # Remove a key-value pair

# Dictionary methods
keys = person.keys()  # dict_keys(['name', 'age', 'email'])
values = person.values()  # dict_values(['Alice', 31, 'alice@example.com'])
items = person.items()  # dict_items([('name', 'Alice'), ('age', 31), ('email', 'alice@example.com')])
```

**Key characteristics:**
- Unordered (in Python < 3.7, ordered in Python >= 3.7)
- Mutable
- Keys must be immutable and unique
- Values can be of any type
- Efficient for lookup, insertion, and deletion by key

## Collections Module

The `collections` module provides specialized data structures that extend the capabilities of the built-in types:

```python
from collections import Counter, defaultdict, namedtuple, deque, OrderedDict

# Counter: counts occurrences of elements
word_counts = Counter("mississippi")  # Counter({'i': 4, 's': 4, 'p': 2, 'm': 1})
most_common = word_counts.most_common(2)  # [('i', 4), ('s', 4)]

# defaultdict: dict with default factory for missing keys
word_groups = defaultdict(list)
for word in ["apple", "banana", "apricot", "berry"]:
    word_groups[word[0]].append(word)
# defaultdict(<class 'list'>, {'a': ['apple', 'apricot'], 'b': ['banana', 'berry']})

# namedtuple: tuple with named fields
Person = namedtuple('Person', ['name', 'age', 'job'])
alice = Person("Alice", 30, "Engineer")
print(alice.name, alice.job)  # Alice Engineer

# deque: efficient double-ended queue
queue = deque(["a", "b", "c"])
queue.append("d")         # Add to right: deque(['a', 'b', 'c', 'd'])
queue.appendleft("z")     # Add to left: deque(['z', 'a', 'b', 'c', 'd'])
queue.pop()               # Remove from right: 'd', queue: deque(['z', 'a', 'b', 'c'])
queue.popleft()           # Remove from left: 'z', queue: deque(['a', 'b', 'c'])

# OrderedDict: remembers insertion order (less relevant since Python 3.7)
ordered = OrderedDict([('a', 1), ('b', 2), ('c', 3)])
```

## Choosing the Right Data Structure

| Data Structure | When to Use |
|----------------|-------------|
| **List** | When you need ordered data that might change |
| **Tuple** | When you need immutable ordered data |
| **Set** | When you need to ensure uniqueness and perform set operations |
| **Dictionary** | When you need key-value mappings with fast lookups |
| **Counter** | When you need to count occurrences of elements |
| **defaultdict** | When you need a dictionary with default values for missing keys |
| **namedtuple** | When you need lightweight, immutable objects with named fields |
| **deque** | When you need efficient appends and pops from both ends |

## Comprehensions

Comprehensions provide a concise way to create data structures from existing iterables:

### List Comprehension

```python
# List comprehension syntax: [expression for item in iterable if condition]

# Creating a list of squares
squares = [x**2 for x in range(10)]  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# Filtering with a condition
even_squares = [x**2 for x in range(10) if x % 2 == 0]  # [0, 4, 16, 36, 64]

# Nested list comprehension
matrix = [[i*j for j in range(3)] for i in range(3)]
# [[0, 0, 0], [0, 1, 2], [0, 2, 4]]
```

### Dictionary Comprehension

```python
# Dictionary comprehension syntax: {key_expr: value_expr for item in iterable if condition}

# Creating a dictionary from two lists
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]
name_to_age = {name: age for name, age in zip(names, ages)}
# {'Alice': 25, 'Bob': 30, 'Charlie': 35}

# Filtering with a condition
squares_dict = {x: x**2 for x in range(10) if x % 2 == 0}
# {0: 0, 2: 4, 4: 16, 6: 36, 8: 64}
```

### Set Comprehension

```python
# Set comprehension syntax: {expression for item in iterable if condition}

# Creating a set of squares
squares_set = {x**2 for x in range(10)}  # {0, 1, 4, 9, 16, 25, 36, 49, 64, 81}

# Filtering with a condition
vowels = {char for char in "hello world" if char in "aeiou"}  # {'e', 'o'}
```

## Deep Dive Sub-topics

For a more detailed exploration of each data structure, refer to the following sub-topics:

- [Lists in Detail](./04a-lists.md)
- [Tuples in Detail](./04b-tuples.md)
- [Sets in Detail](./04c-sets.md)
- [Dictionaries in Detail](./04d-dictionaries.md)
- [Collections Module in Detail](./04e-collections.md)
- [Advanced Comprehensions](./04f-comprehensions.md)

---

[<- Back to Control Flow](./03-control-flow.md) | [Next: Functions ->](./05-functions.md)
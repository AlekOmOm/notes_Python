# 4a. Python Lists 📋

[<- Back to Data Structures](./04-data-structures.md) | [Next Sub-Topic: Tuples ->](./04b-tuples.md)

## Overview

Lists are one of Python's most versatile and commonly used data structures. A list is an ordered, mutable collection that can contain elements of different types, including other lists. Lists are defined by enclosing elements in square brackets `[]`, with elements separated by commas.

## Key Concepts

### Creating Lists

There are multiple ways to create lists in Python:

```python
# Empty list
empty_list = []
also_empty = list()

# List with initial values
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True]

# List from other iterables
chars = list("hello")  # ['h', 'e', 'l', 'l', 'o']
range_list = list(range(5))  # [0, 1, 2, 3, 4]

# Nested lists
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
```

### Accessing List Elements

Python uses zero-based indexing, and supports negative indices to access elements from the end:

```python
fruits = ["apple", "banana", "cherry", "date", "elderberry"]

# Positive indexing (from the beginning)
first_fruit = fruits[0]  # "apple"
third_fruit = fruits[2]  # "cherry"

# Negative indexing (from the end)
last_fruit = fruits[-1]  # "elderberry"
second_last = fruits[-2]  # "date"

# Accessing nested list elements
nested = [[1, 2, 3], [4, 5, 6]]
element = nested[0][1]  # 2
```

### List Slicing

Slicing extracts a portion of a list using the syntax `list[start:stop:step]`:

```python
numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# Basic slicing [start:stop] (stop is exclusive)
first_three = numbers[0:3]  # [0, 1, 2]
middle = numbers[3:7]       # [3, 4, 5, 6]

# Omitting start/stop
from_beginning = numbers[:5]  # [0, 1, 2, 3, 4]
to_end = numbers[5:]          # [5, 6, 7, 8, 9]

# Using step
every_second = numbers[::2]   # [0, 2, 4, 6, 8]
every_third = numbers[::3]    # [0, 3, 6, 9]

# Negative step (reverses the list)
reversed_list = numbers[::-1]  # [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]

# Complex slicing
reversed_section = numbers[7:2:-1]  # [7, 6, 5, 4, 3]
```

## Implementation Patterns

### Pattern 1: List Modification

Lists are mutable, allowing for various modification operations:

```python
colors = ["red", "green", "blue"]

# Adding elements
colors.append("yellow")        # ["red", "green", "blue", "yellow"]
colors.insert(1, "orange")     # ["red", "orange", "green", "blue", "yellow"]
colors.extend(["purple", "pink"])  # ["red", "orange", "green", "blue", "yellow", "purple", "pink"]

# Removing elements
colors.remove("orange")   # Removes first occurrence of "orange"
popped = colors.pop()     # Removes and returns the last element ("pink")
popped_index = colors.pop(1)  # Removes and returns element at index 1 ("green")
del colors[0]             # Removes element at index 0 ("red")

# Clearing a list
colors.clear()  # []
```

### Pattern 2: List Operations

Lists support various operations for common tasks:

```python
numbers = [3, 1, 4, 1, 5, 9, 2, 6]

# Sorting
numbers.sort()               # Sorts in-place
sorted_numbers = sorted(numbers)  # Returns a new sorted list

# Reversing
numbers.reverse()            # Reverses in-place
reversed_list = numbers[::-1]     # Creates a new reversed list
reversed_iter = reversed(numbers)  # Returns an iterator

# Finding elements
index = numbers.index(5)     # Returns index of first occurrence of 5
count = numbers.count(1)     # Counts occurrences of 1
contains = 7 in numbers      # Checks if 7 is in the list (False)

# Min, max, sum
minimum = min(numbers)
maximum = max(numbers)
total = sum(numbers)
```

## Common Challenges and Solutions

### Challenge 1: Modifying Lists While Iterating

Modifying a list while iterating can lead to unexpected behavior:

**Problem:**
```python
numbers = [1, 2, 3, 4, 5]
for num in numbers:
    if num % 2 == 0:
        numbers.remove(num)  # This can cause items to be skipped!
```

**Solution:**
```python
# Method 1: Iterate over a copy
numbers = [1, 2, 3, 4, 5]
for num in numbers[:]:  # Creates a copy for iteration
    if num % 2 == 0:
        numbers.remove(num)

# Method 2: Build a new list (preferred)
numbers = [1, 2, 3, 4, 5]
odds = [num for num in numbers if num % 2 != 0]
# or
odds = list(filter(lambda x: x % 2 != 0, numbers))
```

### Challenge 2: Copying Lists

Simple assignment creates references, not copies:

**Problem:**
```python
original = [1, 2, 3]
copy = original  # This doesn't create a copy!
copy.append(4)   # Modifies both 'copy' and 'original'
print(original)  # [1, 2, 3, 4]
```

**Solution:**
```python
# Shallow copy methods
copy1 = original[:]  # Slice copy
copy2 = list(original)  # Constructor copy
copy3 = original.copy()  # Method copy
import copy
copy4 = copy.copy(original)  # copy module shallow copy

# Deep copy (for nested lists)
import copy
nested = [[1, 2], [3, 4]]
deep_copy = copy.deepcopy(nested)
```

## Practical Example

Here's a complete example that demonstrates many list techniques by implementing a simple to-do list:

```python
def todo_list_manager():
    """A simple to-do list implementation using Python lists."""
    tasks = []
    
    while True:
        print("\n==== To-Do List Manager ====")
        print(f"You have {len(tasks)} tasks in your list.")
        print("1. Add a task")
        print("2. View all tasks")
        print("3. Mark task as complete")
        print("4. Remove a task")
        print("5. Sort tasks")
        print("6. Exit")
        
        choice = input("Enter your choice (1-6): ")
        
        if choice == '1':
            task = input("Enter a new task: ")
            priority = input("Enter priority (High/Medium/Low): ").lower()
            tasks.append({"task": task, "priority": priority, "completed": False})
            print(f"Added: {task}")
            
        elif choice == '2':
            if not tasks:
                print("No tasks in your list!")
            else:
                print("\nYour Tasks:")
                for i, task in enumerate(tasks):
                    status = "✓" if task["completed"] else " "
                    print(f"{i+1}. [{status}] {task['task']} ({task['priority']})")
                    
        elif choice == '3':
            if not tasks:
                print("No tasks to mark as complete!")
            else:
                try:
                    task_num = int(input("Enter task number to mark as complete: ")) - 1
                    if 0 <= task_num < len(tasks):
                        tasks[task_num]["completed"] = True
                        print(f"Marked '{tasks[task_num]['task']}' as complete!")
                    else:
                        print("Invalid task number.")
                except ValueError:
                    print("Please enter a valid number.")
                    
        elif choice == '4':
            if not tasks:
                print("No tasks to remove!")
            else:
                try:
                    task_num = int(input("Enter task number to remove: ")) - 1
                    if 0 <= task_num < len(tasks):
                        removed = tasks.pop(task_num)
                        print(f"Removed: {removed['task']}")
                    else:
                        print("Invalid task number.")
                except ValueError:
                    print("Please enter a valid number.")
                    
        elif choice == '5':
            if not tasks:
                print("No tasks to sort!")
            else:
                print("Sort by:")
                print("1. Priority")
                print("2. Completion status")
                sort_choice = input("Enter sort option (1-2): ")
                
                if sort_choice == '1':
                    # Custom sort order for priorities
                    priority_order = {"high": 0, "medium": 1, "low": 2}
                    tasks.sort(key=lambda x: priority_order.get(x["priority"], 3))
                    print("Tasks sorted by priority!")
                    
                elif sort_choice == '2':
                    # Sort by completion status (completed tasks at the bottom)
                    tasks.sort(key=lambda x: x["completed"])
                    print("Tasks sorted by completion status!")
                    
                else:
                    print("Invalid sort option.")
                    
        elif choice == '6':
            print("Goodbye!")
            break
            
        else:
            print("Invalid choice. Please try again.")

# Run the to-do list manager
if __name__ == "__main__":
    todo_list_manager()
```

## Summary

1. Lists are ordered, mutable collections that can store elements of different types
2. They provide versatile methods for adding, removing, and finding elements
3. Lists support indexing, slicing, and various operations like sorting and counting
4. When modifying lists during iteration, use a copy or create a new list
5. For copying lists, be aware of the difference between shallow and deep copies
6. Lists are a fundamental data structure in Python and form the basis for many algorithms and data processing techniques

---

[<- Back to Data Structures](./04-data-structures.md) | [Next Sub-Topic: Tuples ->](./04b-tuples.md)
# 9. Error Handling 🛡️

[<- Back: File Handling and I/O](./08-file-handling.md) | [Next: Advanced Python Concepts ->](./10-advanced-concepts.md)

## Table of Contents

- [Exception Basics](#exception-basics)
- [Handling Exceptions](#handling-exceptions)
- [Custom Exceptions](#custom-exceptions)
- [Context Managers](#context-managers)
- [Debugging Techniques](#debugging-techniques)
- [Best Practices](#best-practices)

## Exception Basics

Exceptions are events that occur during program execution that disrupt the normal flow of instructions.

### Python's Exception Hierarchy

```
BaseException
 ├── SystemExit                  # Raised by sys.exit()
 ├── KeyboardInterrupt           # Raised by Ctrl+C
 ├── GeneratorExit               # Raised when a generator is closed
 └── Exception                   # Base class for all non-exit exceptions
      ├── StopIteration          # Raised when an iterator has no more items
      ├── ArithmeticError        # Base for arithmetic errors
      │    ├── FloatingPointError
      │    ├── OverflowError
      │    └── ZeroDivisionError
      ├── AssertionError         # Raised by failed assert statements
      ├── AttributeError         # Raised when attribute reference fails
      ├── BufferError            # Related to buffer operations
      ├── EOFError               # Raised when input() hits EOF
      ├── ImportError            # Raised when import fails
      │    └── ModuleNotFoundError
      ├── LookupError            # Base for invalid lookups
      │    ├── IndexError        # Raised when index out of range
      │    └── KeyError          # Raised when key not found in dict
      ├── MemoryError            # Raised when out of memory
      ├── NameError              # Raised when name not found
      │    └── UnboundLocalError
      ├── OSError                # Operating system errors
      │    ├── BlockingIOError
      │    ├── ChildProcessError
      │    ├── ConnectionError
      │    │    ├── BrokenPipeError
      │    │    ├── ConnectionAbortedError
      │    │    ├── ConnectionRefusedError
      │    │    └── ConnectionResetError
      │    ├── FileExistsError
      │    ├── FileNotFoundError
      │    ├── InterruptedError
      │    ├── IsADirectoryError
      │    ├── NotADirectoryError
      │    ├── PermissionError
      │    ├── ProcessLookupError
      │    └── TimeoutError
      ├── ReferenceError         # Raised for weak references
      ├── RuntimeError           # Generic runtime errors
      │    ├── NotImplementedError
      │    └── RecursionError
      ├── SyntaxError            # Raised for syntax errors
      │    └── IndentationError
      │         └── TabError
      ├── SystemError            # Internal interpreter error
      ├── TypeError              # Type error in operations
      ├── ValueError             # Error in value
      │    └── UnicodeError
      │         ├── UnicodeDecodeError
      │         ├── UnicodeEncodeError
      │         └── UnicodeTranslateError
      └── Warning                # Base for warnings
           ├── DeprecationWarning
           ├── FutureWarning
           ├── ImportWarning
           ├── PendingDeprecationWarning
           ├── ResourceWarning
           ├── RuntimeWarning
           ├── SyntaxWarning
           ├── UnicodeWarning
           └── UserWarning
```

## Handling Exceptions

### Basic Try-Except Block

```python
try:
    # Code that might raise an exception
    x = 1 / 0
except ZeroDivisionError:
    # Executed if ZeroDivisionError occurs
    print("Cannot divide by zero!")
```

### Multiple Except Blocks

```python
try:
    num = int(input("Enter a number: "))
    result = 100 / num
    print(f"Result: {result}")
except ValueError:
    print("That's not a valid number!")
except ZeroDivisionError:
    print("Cannot divide by zero!")
```

### Handling Multiple Exceptions in One Block

```python
try:
    # Code that might raise different exceptions
    num = int(input("Enter a number: "))
    result = 100 / num
except (ValueError, ZeroDivisionError):
    print("Invalid input!")
```

### Capturing Exception Information

```python
try:
    # Code that might raise an exception
    open("nonexistent_file.txt")
except FileNotFoundError as e:
    print(f"Error: {e}")  # Access exception details
    print(f"Error type: {type(e).__name__}")
```

### The Else Clause

Executed only if no exceptions were raised in the try block.

```python
try:
    num = int(input("Enter a number: "))
except ValueError:
    print("That's not a valid number!")
else:
    # This runs only if no exceptions occurred in the try block
    print(f"You entered {num}")
```

### The Finally Clause

Always executed, regardless of whether an exception occurred.

```python
try:
    file = open("file.txt", "r")
    content = file.read()
except FileNotFoundError:
    print("File not found!")
finally:
    # This block always executes
    print("Execution completed")
    # Always close resources, even if an exception occurred
    if 'file' in locals() and not file.closed:
        file.close()
```

### Exception Chaining

```python
try:
    # Some code that might raise an exception
    int("not_a_number")
except ValueError as e:
    # Raise a new exception with the original one as context
    raise RuntimeError("Processing error") from e
```

## Custom Exceptions

Creating your own exception types helps make your code more readable and maintainable.

### Defining Custom Exceptions

```python
class ValidationError(Exception):
    """Exception raised for validation errors."""
    
    def __init__(self, message, value):
        self.message = message
        self.value = value
        super().__init__(self.message)
    
    def __str__(self):
        return f"{self.message}: {self.value}"

# Using the custom exception
def validate_age(age):
    if age < 0:
        raise ValidationError("Age cannot be negative", age)
    if age > 120:
        raise ValidationError("Age unrealistically high", age)
    return age

# Handle the custom exception
try:
    validate_age(-5)
except ValidationError as e:
    print(e)  # Age cannot be negative: -5
```

### Extending Built-in Exceptions

```python
class NetworkError(IOError):
    """Exception raised for network-related errors."""
    pass

class DatabaseError(Exception):
    """Base class for database-related errors."""
    pass

class QueryError(DatabaseError):
    """Exception raised for query errors."""
    
    def __init__(self, query, message):
        self.query = query
        self.message = message
        super().__init__(message)
```

### Custom Exception Hierarchy

```python
class ApplicationError(Exception):
    """Base exception for the application."""
    pass

class ConfigurationError(ApplicationError):
    """Configuration-related errors."""
    pass

class DatabaseError(ApplicationError):
    """Database-related errors."""
    pass

class APIError(ApplicationError):
    """API-related errors."""
    
    def __init__(self, status_code, message):
        self.status_code = status_code
        self.message = message
        super().__init__(f"API Error ({status_code}): {message}")
```

## Context Managers

Context managers help manage resources properly, automatically handling cleanup.

### Using with Statement

```python
# File handling with context manager
with open("file.txt", "r") as file:
    content = file.read()
    # File is automatically closed when the block exits

# Multiple context managers
with open("input.txt", "r") as input_file, open("output.txt", "w") as output_file:
    for line in input_file:
        output_file.write(line.upper())
    # Both files are automatically closed
```

### Creating Context Managers with Classes

```python
class FileManager:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = None
        
    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
        # Return True to suppress any exception
        # Return False or None to propagate exceptions
        return False

# Using the custom context manager
with FileManager("file.txt", "r") as file:
    content = file.read()
```

### Context Managers with contextlib

```python
from contextlib import contextmanager

@contextmanager
def file_manager(filename, mode):
    try:
        file = open(filename, mode)
        yield file
    finally:
        file.close()

# Using the decorator-based context manager
with file_manager("file.txt", "r") as file:
    content = file.read()
```

## Debugging Techniques

### Using print() Statements

```python
def buggy_function(data):
    print(f"Input data: {data}")  # Debug input
    
    result = []
    for i, item in enumerate(data):
        print(f"Processing item {i}: {item}")  # Debug processing
        
        # Some complex operation
        value = item * 2
        print(f"Calculated value: {value}")  # Debug output
        
        result.append(value)
    
    print(f"Final result: {result}")  # Debug result
    return result
```

### Using the assert Statement

```python
def calculate_average(numbers):
    assert len(numbers) > 0, "Cannot calculate average of empty list"
    total = sum(numbers)
    return total / len(numbers)

# Will raise AssertionError if condition fails
avg = calculate_average([1, 2, 3])  # Works fine
# avg = calculate_average([])  # AssertionError: Cannot calculate average of empty list
```

### Using the logging Module

```python
import logging

# Configure the logging system
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    filename='app.log'
)

# Create a logger
logger = logging.getLogger('my_app')

def risky_function(value):
    logger.debug(f"risky_function called with value: {value}")
    
    try:
        result = 100 / value
        logger.info(f"Calculation successful, result: {result}")
        return result
    except ZeroDivisionError:
        logger.error("Division by zero attempted")
        raise
    finally:
        logger.debug("risky_function completed")

# Different log levels
logger.debug("Detailed information for debugging")
logger.info("Confirmation that things are working")
logger.warning("Something unexpected happened")
logger.error("A more serious problem")
logger.critical("A very serious error")
```

### Using the debugger (pdb)

```python
def complex_function(a, b):
    import pdb; pdb.set_trace()  # Debugger will start here
    result = a * b
    for i in range(a):
        result += i
    return result

# Common pdb commands:
# c: continue execution
# n: next line (step over)
# s: step into function
# r: return from current function
# p expr: evaluate expression
# l: list source code
# q: quit debugger
```

### Using the breakpoint() Function (Python 3.7+)

```python
def complex_function(a, b):
    breakpoint()  # Shorthand for import pdb; pdb.set_trace()
    result = a * b
    for i in range(a):
        result += i
    return result
```

## Best Practices

### Use Specific Exceptions

```python
# Bad: Catches all exceptions, masks bugs
try:
    # Code that might raise various exceptions
    file = open("config.txt")
    config = json.loads(file.read())
    value = 100 / config["divisor"]
except Exception as e:
    print(f"An error occurred: {e}")

# Good: Use specific exception handlers
try:
    file = open("config.txt")
    config = json.loads(file.read())
    value = 100 / config["divisor"]
except FileNotFoundError:
    print("Config file not found, using defaults")
except json.JSONDecodeError:
    print("Config file is not valid JSON")
except KeyError:
    print("Config file is missing required fields")
except ZeroDivisionError:
    print("Divisor cannot be zero")
```

### Avoid Bare except Clauses

```python
# Bad: Catches even keyboard interrupts and system exits
try:
    risky_operation()
except:  # Bare except clause
    print("Error occurred")

# Good: Use Exception to avoid catching system exits
try:
    risky_operation()
except Exception as e:
    print(f"Error occurred: {e}")
```

### Clean Up Resources Properly

```python
# Bad: Resources might not be closed if exceptions occur
file = open("file.txt", "r")
data = file.read()
file.close()

# Good: Use context managers for resource management
with open("file.txt", "r") as file:
    data = file.read()
# File is automatically closed, even if exceptions occur
```

### Don't Silence Exceptions Without Good Reason

```python
# Bad: Silently ignoring exceptions
try:
    process_data()
except Exception:
    pass  # Silently ignoring all errors

# Good: Handle or log exceptions properly
try:
    process_data()
except Exception as e:
    logger.error(f"Failed to process data: {e}")
    # Optionally re-raise or handle the error appropriately
```

### Include Contextual Information

```python
def process_user_data(user_id, data):
    try:
        # Process the data
        update_database(user_id, data)
    except DatabaseError as e:
        # Include context in the exception
        raise DatabaseError(f"Failed to update user {user_id}: {e}") from e
```

### Use Custom Exceptions for Domain Logic

```python
class InsufficientFundsError(Exception):
    def __init__(self, account, amount_needed, amount_available):
        self.account = account
        self.amount_needed = amount_needed
        self.amount_available = amount_available
        message = f"Account {account} needs {amount_needed} but only has {amount_available}"
        super().__init__(message)

def withdraw(account, amount):
    balance = get_balance(account)
    if balance < amount:
        raise InsufficientFundsError(account, amount, balance)
    # Process withdrawal
```

---

[<- Back: File Handling and I/O](./08-file-handling.md) | [Next: Advanced Python Concepts ->](./10-advanced-concepts.md)

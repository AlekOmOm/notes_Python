# 7b. Testing Object-Oriented Python 🧪

[<- Back to Object-Oriented Programming](./07-oop.md) | [Next Sub-Topic: Dataclass Decorator ->](./07a-dataclass-decorator.md)

## Overview

Testing is a critical part of software development. This sub-topic covers how to create and organize tests for object-oriented Python code, focusing on testing classes, methods, properties, and interactions between objects. We'll explore testing frameworks, methodologies, and best practices specific to OOP testing.

## Testing Frameworks

Python offers several testing frameworks, each with different strengths:

### unittest (Standard Library)

```python
import unittest

class TestMyClass(unittest.TestCase):
    def setUp(self):
        # Runs before each test
        self.my_object = MyClass()
    
    def test_some_method(self):
        result = self.my_object.some_method()
        self.assertEqual(result, expected_value)
    
    def tearDown(self):
        # Runs after each test
        pass

if __name__ == '__main__':
    unittest.main()
```

### pytest (Third-party, Recommended)

```python
import pytest

# Simple function test
def test_my_function():
    result = my_function()
    assert result == expected_value

# Class test with fixture
@pytest.fixture
def my_object():
    return MyClass()

def test_some_method(my_object):
    result = my_object.some_method()
    assert result == expected_value
```

## Testing Classes

Testing object-oriented code requires testing:
1. Object initialization
2. Method behavior
3. Properties and attributes
4. Inheritance relationships
5. Composition and interactions

### Testing Object Initialization

```python
# Class to test
class Person:
    def __init__(self, name, age):
        if age < 0:
            raise ValueError("Age cannot be negative")
        self.name = name
        self.age = age

# unittest approach
class TestPerson(unittest.TestCase):
    def test_initialization(self):
        person = Person("Alice", 30)
        self.assertEqual(person.name, "Alice")
        self.assertEqual(person.age, 30)
        
    def test_negative_age(self):
        with self.assertRaises(ValueError):
            Person("Bob", -5)

# pytest approach
def test_person_initialization():
    person = Person("Alice", 30)
    assert person.name == "Alice"
    assert person.age == 30
    
def test_person_negative_age():
    with pytest.raises(ValueError):
        Person("Bob", -5)
```

### Testing Methods

```python
class Calculator:
    def __init__(self, initial_value=0):
        self.value = initial_value
    
    def add(self, x):
        self.value += x
        return self.value
        
    def multiply(self, x):
        self.value *= x
        return self.value

# Testing instance methods
def test_calculator_methods():
    calc = Calculator(5)
    
    # Test method return value
    assert calc.add(3) == 8
    
    # Test state change
    assert calc.value == 8
    
    # Test chained operations
    assert calc.multiply(2) == 16
    assert calc.value == 16
```

### Testing Properties

```python
class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius
    
    @property
    def celsius(self):
        return self._celsius
    
    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Temperature below absolute zero")
        self._celsius = value
    
    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32

# Testing properties
def test_temperature_properties():
    temp = Temperature(25)
    
    # Test getter
    assert temp.celsius == 25
    
    # Test derived property
    assert temp.fahrenheit == 77
    
    # Test setter
    temp.celsius = 30
    assert temp.celsius == 30
    assert temp.fahrenheit == 86
    
    # Test validation
    with pytest.raises(ValueError):
        temp.celsius = -300
```

## Implementation Patterns

### Pattern 1: Test Fixture Setup

```python
# Using pytest fixtures for common setup
@pytest.fixture
def bank_account():
    """Fixture that creates a bank account with a $100 balance."""
    from banking import BankAccount
    account = BankAccount("12345", initial_balance=100)
    return account

def test_deposit(bank_account):
    bank_account.deposit(50)
    assert bank_account.balance == 150

def test_withdrawal(bank_account):
    bank_account.withdraw(30)
    assert bank_account.balance == 70
```

**When to use this pattern:**
- When multiple tests need the same initial object state
- For complex object setup that would be repetitive
- When setup resources need to be properly managed

### Pattern 2: Mocking Dependencies

```python
from unittest.mock import Mock, patch

class UserService:
    def __init__(self, database):
        self.database = database
    
    def get_user(self, user_id):
        return self.database.find_user(user_id)
    
    def create_user(self, name, email):
        if self.database.find_user_by_email(email):
            raise ValueError("Email already exists")
        return self.database.create_user(name, email)

# Test with mock
def test_get_user():
    # Create a mock database
    mock_db = Mock()
    mock_db.find_user.return_value = {"id": 123, "name": "Test User"}
    
    # Inject the mock into the service
    service = UserService(mock_db)
    user = service.get_user(123)
    
    # Verify result
    assert user["name"] == "Test User"
    
    # Verify interaction with the mock
    mock_db.find_user.assert_called_once_with(123)

# Test with patch
@patch("my_module.Database")
def test_create_user(mock_database_class):
    # Configure the mock
    mock_db = mock_database_class.return_value
    mock_db.find_user_by_email.return_value = None
    mock_db.create_user.return_value = {"id": 1, "name": "New User", "email": "new@example.com"}
    
    # Create service with the mocked database
    from my_module import UserService
    service = UserService(mock_db)
    
    # Test method
    user = service.create_user("New User", "new@example.com")
    
    # Verify result
    assert user["name"] == "New User"
    
    # Verify interactions
    mock_db.find_user_by_email.assert_called_once_with("new@example.com")
    mock_db.create_user.assert_called_once_with("New User", "new@example.com")
```

**When to use this pattern:**
- When testing classes that depend on external systems
- To isolate the unit under test
- To test error handling and edge cases that are hard to trigger with real dependencies

### Pattern 3: Parameterized Testing

```python
@pytest.mark.parametrize("input_value,expected", [
    (5, 10),
    (0, 0),
    (-5, -10),
    (3.5, 7)
])
def test_double_function(input_value, expected):
    from my_module import double
    assert double(input_value) == expected

# For class methods
class ShapeCalculator:
    def calculate_area(self, shape_type, dimensions):
        if shape_type == "rectangle":
            return dimensions["length"] * dimensions["width"]
        elif shape_type == "circle":
            import math
            return math.pi * dimensions["radius"] ** 2
        else:
            raise ValueError(f"Unknown shape: {shape_type}")

@pytest.mark.parametrize("shape_type,dimensions,expected", [
    ("rectangle", {"length": 4, "width": 5}, 20),
    ("circle", {"radius": 2}, 12.56637),  # Using approximate value
    ("rectangle", {"length": 0, "width": 5}, 0)
])
def test_area_calculation(shape_type, dimensions, expected):
    calculator = ShapeCalculator()
    result = calculator.calculate_area(shape_type, dimensions)
    assert result == pytest.approx(expected)
```

**When to use this pattern:**
- When testing functions or methods with multiple input-output combinations
- For comprehensive testing of edge cases
- When testing numerical algorithms that need approximation checks

## Common Challenges and Solutions

### Challenge 1: Testing Inheritance

**Solution:**
```python
# Base class
class Animal:
    def __init__(self, name):
        self.name = name
    
    def make_sound(self):
        return "..."

# Derived class
class Dog(Animal):
    def make_sound(self):
        return "Woof!"

# Test both the inherited behavior and overridden behavior
def test_animal_inheritance():
    # Test the base class
    animal = Animal("Generic Animal")
    assert animal.name == "Generic Animal"
    assert animal.make_sound() == "..."
    
    # Test the derived class
    dog = Dog("Rex")
    # Test inherited attribute
    assert dog.name == "Rex"
    # Test overridden method
    assert dog.make_sound() == "Woof!"
    
    # Test polymorphism
    def get_sound(animal_obj):
        return animal_obj.make_sound()
    
    assert get_sound(animal) == "..."
    assert get_sound(dog) == "Woof!"
```

### Challenge 2: Testing with External State

**Solution:**
```python
# Class with external state
class Logger:
    def __init__(self, log_file):
        self.log_file = log_file
    
    def log(self, message):
        with open(self.log_file, 'a') as f:
            f.write(f"{message}\n")
    
    def read_logs(self):
        try:
            with open(self.log_file, 'r') as f:
                return f.read()
        except FileNotFoundError:
            return ""

# Test using temporary files
import tempfile
import os

def test_logger():
    # Create a temporary file for testing
    with tempfile.NamedTemporaryFile(delete=False) as temp:
        temp_name = temp.name
    
    try:
        # Use the temp file for the logger
        logger = Logger(temp_name)
        
        # Test logging
        logger.log("Test message 1")
        logger.log("Test message 2")
        
        # Test reading
        logs = logger.read_logs()
        assert "Test message 1" in logs
        assert "Test message 2" in logs
    finally:
        # Clean up the temporary file
        os.unlink(temp_name)
```

## Practical Example

Here's a complete example showing how to test a `BankAccount` class with various testing approaches:

```python
# bank_account.py
class InsufficientFundsError(Exception):
    pass

class BankAccount:
    def __init__(self, account_number, initial_balance=0):
        self.account_number = account_number
        self._balance = initial_balance
        self._transactions = []
        
        if initial_balance > 0:
            self._transactions.append(("deposit", initial_balance))
    
    @property
    def balance(self):
        return self._balance
    
    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        
        self._balance += amount
        self._transactions.append(("deposit", amount))
        return self._balance
    
    def withdraw(self, amount):
        if amount <= 0:
            raise ValueError("Withdrawal amount must be positive")
        
        if amount > self._balance:
            raise InsufficientFundsError(f"Cannot withdraw ${amount}. Current balance: ${self._balance}")
        
        self._balance -= amount
        self._transactions.append(("withdrawal", amount))
        return self._balance
    
    def get_transaction_history(self):
        return self._transactions.copy()

# test_bank_account.py
import pytest
from bank_account import BankAccount, InsufficientFundsError

@pytest.fixture
def account():
    """Create a bank account with $100 initial balance."""
    return BankAccount("12345", 100)

def test_initialization():
    # Test default initialization
    account1 = BankAccount("11111")
    assert account1.account_number == "11111"
    assert account1.balance == 0
    assert account1.get_transaction_history() == []
    
    # Test initialization with balance
    account2 = BankAccount("22222", 200)
    assert account2.account_number == "22222"
    assert account2.balance == 200
    assert account2.get_transaction_history() == [("deposit", 200)]

def test_deposit(account):
    # Test successful deposit
    new_balance = account.deposit(50)
    assert new_balance == 150
    assert account.balance == 150
    
    # Verify transaction history
    history = account.get_transaction_history()
    assert len(history) == 2
    assert history[0] == ("deposit", 100)  # Initial deposit
    assert history[1] == ("deposit", 50)   # Our new deposit
    
    # Test invalid deposit
    with pytest.raises(ValueError) as exc_info:
        account.deposit(-50)
    assert "Deposit amount must be positive" in str(exc_info.value)

def test_withdraw(account):
    # Test successful withdrawal
    new_balance = account.withdraw(30)
    assert new_balance == 70
    assert account.balance == 70
    
    # Verify transaction history
    history = account.get_transaction_history()
    assert len(history) == 2
    assert history[1] == ("withdrawal", 30)
    
    # Test withdrawal exceeding balance
    with pytest.raises(InsufficientFundsError) as exc_info:
        account.withdraw(100)
    assert "Cannot withdraw $100" in str(exc_info.value)
    
    # Test invalid withdrawal amount
    with pytest.raises(ValueError) as exc_info:
        account.withdraw(0)
    assert "Withdrawal amount must be positive" in str(exc_info.value)

@pytest.mark.parametrize("initial_balance,deposit_amount,withdrawal_amount,expected_balance", [
    (100, 50, 30, 120),
    (0, 200, 50, 150),
    (1000, 0, 500, 500)
])
def test_transaction_sequence(initial_balance, deposit_amount, withdrawal_amount, expected_balance):
    # Test a sequence of operations
    account = BankAccount("test", initial_balance)
    
    if deposit_amount > 0:
        account.deposit(deposit_amount)
    
    if withdrawal_amount > 0:
        account.withdraw(withdrawal_amount)
    
    assert account.balance == expected_balance

def test_transaction_history_isolation(account):
    # Get history
    history1 = account.get_transaction_history()
    
    # Modify the returned list
    history1.append(("fake", 1000))
    
    # Get history again
    history2 = account.get_transaction_history()
    
    # Verify that the internal list wasn't modified
    assert len(history2) == 1
    assert history2[0] == ("deposit", 100)
```

## Summary

Testing object-oriented Python code requires:

1. **Testing class initialization**: Verify object construction with different parameters
2. **Testing instance methods**: Check both return values and state changes
3. **Testing property behavior**: Validate getters, setters, and computed properties
4. **Testing inheritance**: Verify both inherited and overridden behavior
5. **Using fixtures**: Set up common test objects efficiently
6. **Mocking dependencies**: Isolate classes from external systems
7. **Parameterized tests**: Test multiple scenarios efficiently
8. **Managing external state**: Handle files and other resources safely

Remember that good tests should be:
- **Isolated**: Tests shouldn't depend on each other
- **Repeatable**: Same results every time
- **Fast**: Quick to run for frequent execution
- **Comprehensive**: Cover normal cases, edge cases, and error conditions
- **Readable**: Clear what's being tested and what the expected outcomes are

Effective testing builds confidence in your code's correctness, facilitates refactoring, and serves as executable documentation of how your classes should behave.

---

[<- Back to Object-Oriented Programming](./07-oop.md) | [Next Sub-Topic: Dataclass Decorator ->](./07a-dataclass-decorator.md)

# 12. Testing and Documentation 🧪

[<- Back: Concurrency and Parallelism](./11-concurrency.md) | [Next: Data Science and Analysis ->](./13-data-science.md)

## Table of Contents

- [Unit Testing](#unit-testing)
- [Integration Testing](#integration-testing)
- [Test-Driven Development](#test-driven-development)
- [Documentation Standards](#documentation-standards)
- [Testing Best Practices](#testing-best-practices)
- [Code Quality Tools](#code-quality-tools)

## Unit Testing

Unit testing involves testing individual components of your code in isolation to ensure they function correctly.

### unittest Framework

Python's built-in `unittest` framework provides a solid foundation for testing.

```python
import unittest

def add(a, b):
    return a + b

class TestAddFunction(unittest.TestCase):
    def test_add_positive_numbers(self):
        self.assertEqual(add(1, 2), 3)
        
    def test_add_negative_numbers(self):
        self.assertEqual(add(-1, -1), -2)
        
    def test_add_mixed_numbers(self):
        self.assertEqual(add(-1, 1), 0)
        
    def test_add_zeros(self):
        self.assertEqual(add(0, 0), 0)

if __name__ == '__main__':
    unittest.main()
```

### Test Fixtures and Setup/Teardown

```python
import unittest
import tempfile
import os

class TestFileOperations(unittest.TestCase):
    def setUp(self):
        # This runs before each test
        self.test_dir = tempfile.mkdtemp()
        self.test_file_path = os.path.join(self.test_dir, "test_file.txt")
        
        # Create a test file
        with open(self.test_file_path, 'w') as f:
            f.write("Test content")
    
    def tearDown(self):
        # This runs after each test
        os.remove(self.test_file_path)
        os.rmdir(self.test_dir)
    
    def test_file_exists(self):
        self.assertTrue(os.path.exists(self.test_file_path))
    
    def test_file_content(self):
        with open(self.test_file_path, 'r') as f:
            content = f.read()
        self.assertEqual(content, "Test content")
        
    # Alternative with class-level setup (for all tests in class)
    @classmethod
    def setUpClass(cls):
        # This runs once before all tests in the class
        cls.global_resource = "shared resource"
        
    @classmethod
    def tearDownClass(cls):
        # This runs once after all tests in the class
        cls.global_resource = None
```

### pytest Framework

`pytest` is a more modern and powerful testing framework for Python.

```python
# Install pytest: pip install pytest

# Basic test function - no classes needed
def test_add():
    assert add(1, 2) == 3
    assert add(-1, -1) == -2
    assert add(-1, 1) == 0

# Using fixtures for setup and teardown
import pytest
import tempfile
import os

@pytest.fixture
def temp_file():
    # Setup
    fd, path = tempfile.mkstemp()
    with os.fdopen(fd, 'w') as f:
        f.write("Test content")
    
    # Provide the fixture value
    yield path
    
    # Teardown
    os.unlink(path)

def test_read_file(temp_file):
    with open(temp_file, 'r') as f:
        content = f.read()
    assert content == "Test content"

# Parametrized tests
@pytest.mark.parametrize("a,b,expected", [
    (1, 2, 3),
    (-1, -1, -2),
    (-1, 1, 0),
    (0, 0, 0),
])
def test_add_parametrized(a, b, expected):
    assert add(a, b) == expected
```

### Mocking and Patching

Mocking allows you to replace parts of your system with mock objects for testing.

```python
from unittest import mock
import unittest
import requests

def get_user_data(user_id):
    response = requests.get(f"https://api.example.com/users/{user_id}")
    if response.status_code == 200:
        return response.json()
    else:
        return None

class TestUserData(unittest.TestCase):
    @mock.patch('requests.get')
    def test_get_user_data_success(self, mock_get):
        # Configure the mock
        mock_response = mock.Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = {'id': 123, 'name': 'Test User'}
        mock_get.return_value = mock_response
        
        # Call the function
        result = get_user_data(123)
        
        # Assertions
        mock_get.assert_called_once_with('https://api.example.com/users/123')
        self.assertEqual(result, {'id': 123, 'name': 'Test User'})
    
    @mock.patch('requests.get')
    def test_get_user_data_not_found(self, mock_get):
        # Configure the mock for failure
        mock_response = mock.Mock()
        mock_response.status_code = 404
        mock_get.return_value = mock_response
        
        # Call the function
        result = get_user_data(999)
        
        # Assertions
        self.assertIsNone(result)

# In pytest
def test_get_user_data_with_pytest(monkeypatch):
    # Mock response for requests.get
    class MockResponse:
        def __init__(self, json_data, status_code):
            self.json_data = json_data
            self.status_code = status_code
        
        def json(self):
            return self.json_data
    
    def mock_get(*args, **kwargs):
        return MockResponse({'id': 123, 'name': 'Test User'}, 200)
    
    # Apply the monkeypatch
    monkeypatch.setattr(requests, "get", mock_get)
    
    # Test the function
    result = get_user_data(123)
    assert result == {'id': 123, 'name': 'Test User'}
```

## Integration Testing

Integration testing verifies that different components work together as expected.

### Testing Database Interactions

```python
import unittest
import sqlite3
import os

class UserDatabase:
    def __init__(self, db_path):
        self.db_path = db_path
        self._create_tables()
    
    def _create_tables(self):
        with sqlite3.connect(self.db_path) as conn:
            conn.execute('''
                CREATE TABLE IF NOT EXISTS users (
                    id INTEGER PRIMARY KEY,
                    name TEXT NOT NULL,
                    email TEXT UNIQUE NOT NULL
                )
            ''')
    
    def add_user(self, name, email):
        with sqlite3.connect(self.db_path) as conn:
            try:
                conn.execute(
                    "INSERT INTO users (name, email) VALUES (?, ?)",
                    (name, email)
                )
                return True
            except sqlite3.IntegrityError:
                return False
    
    def get_user_by_email(self, email):
        with sqlite3.connect(self.db_path) as conn:
            conn.row_factory = sqlite3.Row
            cursor = conn.execute(
                "SELECT * FROM users WHERE email = ?",
                (email,)
            )
            row = cursor.fetchone()
            return dict(row) if row else None

class TestUserDatabase(unittest.TestCase):
    def setUp(self):
        self.db_path = "test_users.db"
        self.db = UserDatabase(self.db_path)
    
    def tearDown(self):
        if os.path.exists(self.db_path):
            os.remove(self.db_path)
    
    def test_add_and_get_user(self):
        # Add a user
        result = self.db.add_user("Test User", "test@example.com")
        self.assertTrue(result)
        
        # Get the user
        user = self.db.get_user_by_email("test@example.com")
        self.assertIsNotNone(user)
        self.assertEqual(user["name"], "Test User")
        self.assertEqual(user["email"], "test@example.com")
    
    def test_add_duplicate_email(self):
        # Add a user first time
        result1 = self.db.add_user("User One", "same@example.com")
        self.assertTrue(result1)
        
        # Try to add another user with the same email
        result2 = self.db.add_user("User Two", "same@example.com")
        self.assertFalse(result2)
```

### Testing API Endpoints

```python
# Using pytest and requests to test a REST API
import pytest
import requests

# Constants
API_URL = "https://api.example.com"
HEADERS = {"Content-Type": "application/json"}

# Example: Testing a user API
def test_get_user():
    # Assuming the API requires authentication
    auth = ("username", "password")
    
    # Make the API request
    response = requests.get(f"{API_URL}/users/123", auth=auth)
    
    # Assert status code
    assert response.status_code == 200
    
    # Assert response data
    data = response.json()
    assert data["id"] == 123
    assert "name" in data
    assert "email" in data

def test_create_user():
    # Prepare user data
    user_data = {
        "name": "New User",
        "email": "newuser@example.com"
    }
    
    # Make the API request
    response = requests.post(
        f"{API_URL}/users",
        json=user_data,
        headers=HEADERS
    )
    
    # Assert successful creation
    assert response.status_code == 201
    
    # Assert the created user data
    data = response.json()
    assert data["name"] == user_data["name"]
    assert data["email"] == user_data["email"]
    assert "id" in data
```

## Test-Driven Development

Test-Driven Development (TDD) is a software development process that relies on writing tests before implementing the actual code.

### The TDD Cycle

```
1. Write a failing test for a feature
2. Implement the minimum code to make the test pass
3. Refactor the code while keeping the test passing
4. Repeat
```

### TDD Example

```python
# Step 1: Write a failing test
def test_calculate_total_price():
    cart = ShoppingCart()
    cart.add_item("apple", 1.0, 3)
    cart.add_item("banana", 0.5, 6)
    assert cart.calculate_total() == 6.0  # 3 apples at $1 each + 6 bananas at $0.5 each

# Step 2: Implement the minimum code to make the test pass
class ShoppingCart:
    def __init__(self):
        self.items = []
    
    def add_item(self, name, price, quantity):
        self.items.append({
            "name": name,
            "price": price,
            "quantity": quantity
        })
    
    def calculate_total(self):
        total = 0
        for item in self.items:
            total += item["price"] * item["quantity"]
        return total

# Step 3: Write more tests and expand functionality
def test_remove_item():
    cart = ShoppingCart()
    cart.add_item("apple", 1.0, 3)
    cart.add_item("banana", 0.5, 6)
    cart.remove_item("apple")
    assert cart.calculate_total() == 3.0  # Only bananas left

# Step 4: Implement the new functionality
class ShoppingCart:
    def __init__(self):
        self.items = []
    
    def add_item(self, name, price, quantity):
        self.items.append({
            "name": name,
            "price": price,
            "quantity": quantity
        })
    
    def remove_item(self, name):
        self.items = [item for item in self.items if item["name"] != name]
    
    def calculate_total(self):
        total = 0
        for item in self.items:
            total += item["price"] * item["quantity"]
        return total
```

## Documentation Standards

Good documentation is essential for making your code maintainable and accessible to others.

### Docstrings

Python uses docstrings for documenting modules, classes, and functions.

```python
def calculate_area(length, width):
    """
    Calculate the area of a rectangle.
    
    Args:
        length (float): The length of the rectangle.
        width (float): The width of the rectangle.
    
    Returns:
        float: The area of the rectangle.
    
    Raises:
        ValueError: If length or width is negative.
    
    Examples:
        >>> calculate_area(5, 10)
        50.0
    """
    if length < 0 or width < 0:
        raise ValueError("Length and width must be positive")
    return length * width
```

### Docstring Styles

#### Google Style

```python
def fetch_user(user_id):
    """Fetches user data from the database.
    
    Args:
        user_id (int): The ID of the user to fetch.
        
    Returns:
        dict: A dictionary containing user data with keys:
            - id (int): The user ID
            - name (str): The user's full name
            - email (str): The user's email address
            
    Raises:
        ValueError: If user_id is negative.
        UserNotFoundError: If user cannot be found.
    """
```

#### reStructuredText (Sphinx) Style

```python
def fetch_user(user_id):
    """Fetches user data from the database.
    
    :param user_id: The ID of the user to fetch
    :type user_id: int
    
    :return: A dictionary containing user data
    :rtype: dict
    
    :raises ValueError: If user_id is negative
    :raises UserNotFoundError: If user cannot be found
    """
```

#### NumPy Style

```python
def fetch_user(user_id):
    """
    Fetches user data from the database.
    
    Parameters
    ----------
    user_id : int
        The ID of the user to fetch
    
    Returns
    -------
    dict
        A dictionary containing user data
    
    Raises
    ------
    ValueError
        If user_id is negative
    UserNotFoundError
        If user cannot be found
    """
```

### Type Hints (Python 3.5+)

```python
from typing import Dict, List, Optional, Union, Tuple

def process_data(
    data: List[Dict[str, Union[str, int]]],
    options: Optional[Dict[str, bool]] = None
) -> Tuple[List[int], str]:
    """
    Process a list of data items.
    
    Args:
        data: List of dictionaries containing data to process
        options: Optional configuration for processing
    
    Returns:
        A tuple containing the processed data and a status message
    """
    # Function implementation
    return [1, 2, 3], "Success"
```

## Testing Best Practices

### Test Organization and Naming

```python
# Organization
#
# project/
# ├── src/
# │   └── mypackage/
# │       ├── __init__.py
# │       ├── module1.py
# │       └── module2.py
# └── tests/
#     ├── __init__.py
#     ├── test_module1.py
#     └── test_module2.py

# Naming conventions
def test_should_return_correct_area_for_positive_dimensions():
    # Test implementation

def test_should_raise_exception_for_negative_dimensions():
    # Test implementation
```

### Test Coverage

```python
# Install coverage: pip install coverage
# Run tests with coverage
# coverage run -m pytest
# Generate report
# coverage report
# Generate HTML report
# coverage html

# Example for specific modules
# coverage run -m pytest tests/test_module1.py tests/test_module2.py
# coverage report -m src/mypackage/module1.py src/mypackage/module2.py

# .coveragerc file
# [run]
# source = src/mypackage
# omit = */tests/*
#
# [report]
# exclude_lines =
#     pragma: no cover
#     def __repr__
#     raise NotImplementedError
```

### Testing Edge Cases

```python
def test_division_edge_cases():
    # Test with large numbers
    assert divide(1000000000, 2) == 500000000
    
    # Test with very small numbers
    assert abs(divide(0.1, 3) - 0.03333333) < 1e-8
    
    # Test with zero
    with pytest.raises(ZeroDivisionError):
        divide(5, 0)
    
    # Test with negative numbers
    assert divide(-6, 3) == -2
    assert divide(6, -3) == -2
    assert divide(-6, -3) == 2
```

### Performance Testing

```python
import time
import pytest

def fibonacci_recursive(n):
    if n <= 1:
        return n
    return fibonacci_recursive(n-1) + fibonacci_recursive(n-2)

def fibonacci_iterative(n):
    if n <= 1:
        return n
    a, b = 0, 1
    for _ in range(2, n+1):
        a, b = b, a + b
    return b

@pytest.mark.parametrize("n", [10, 20, 30])
def test_fibonacci_performance(n, benchmark):
    # Using pytest-benchmark plugin
    result = benchmark(fibonacci_iterative, n)
    assert result == fibonacci_recursive(n)

# Manual timing
def test_execution_time():
    n = 30
    
    # Measure iterative version
    start = time.time()
    fibonacci_iterative(n)
    iterative_time = time.time() - start
    
    # Measure recursive version
    start = time.time()
    fibonacci_recursive(n)
    recursive_time = time.time() - start
    
    # Assert recursive is slower for large n
    assert recursive_time > iterative_time
```

## Code Quality Tools

### Linting with Flake8

```python
# Install: pip install flake8
# Usage: flake8 src/

# Configuration (.flake8 file)
# [flake8]
# max-line-length = 88
# exclude = .git,__pycache__,build,dist
# ignore = E203, W503
# per-file-ignores =
#     __init__.py: F401
```

### Code Formatting with Black

```python
# Install: pip install black
# Usage: black src/

# Configuration (pyproject.toml)
# [tool.black]
# line-length = 88
# target-version = ['py38']
# include = '\.pyi?$'
# exclude = '''
# /(
#     \.git
#   | \.hg
#   | \.mypy_cache
#   | \.tox
#   | \.venv
#   | _build
#   | buck-out
#   | build
#   | dist
# )/
# '''
```

### Type Checking with mypy

```python
# Install: pip install mypy
# Usage: mypy src/

# Configuration (mypy.ini)
# [mypy]
# python_version = 3.8
# warn_return_any = True
# warn_unused_configs = True
# disallow_untyped_defs = True
# disallow_incomplete_defs = True
# 
# [mypy.plugins.numpy.*]
# follow_imports = skip
```

### Continuous Integration

```yaml
# GitHub Actions example (.github/workflows/test.yml)
name: Test

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.7, 3.8, 3.9]

    steps:
    - uses: actions/checkout@v2
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v2
      with:
        python-version: ${{ matrix.python-version }}
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install flake8 pytest pytest-cov
        if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
    - name: Lint with flake8
      run: |
        flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
    - name: Test with pytest
      run: |
        pytest --cov=src/ tests/
```

---

[<- Back: Concurrency and Parallelism](./11-concurrency.md) | [Next: Data Science and Analysis ->](./13-data-science.md)

# 13a. NumPy: Numerical Python 🔢

[<- Back to Main Topic](./13-data-science.md) | [Next Sub-Topic: Pandas Data Manipulation ->](./13b-pandas-data-manipulation.md)

## Overview

NumPy (Numerical Python) is the fundamental package for scientific computing in Python. It provides:

*   A powerful N-dimensional array object (`ndarray`).
*   Sophisticated (broadcasting) functions.
*   Tools for integrating C/C++ and Fortran code.
*   Useful linear algebra, Fourier transform, and random number capabilities.

The core strength of NumPy lies in its `ndarray` object, which allows for efficient element-wise operations (vectorization), often much faster than standard Python lists for numerical tasks.

## Key Concepts

### Import Convention

```python
import numpy as np
```

### Array Creation

```python
# From Python lists
a = np.array([1, 2, 3, 4, 5])

# 2D array
b = np.array([[1, 2, 3], [4, 5, 6]])

# Using functions
np.arange(0, 10, 2)     # [0, 2, 4, 6, 8]
np.linspace(0, 1, 5)    # 5 points from 0 to 1
np.zeros((3, 3))        # 3x3 array of zeros
np.ones((2, 4))         # 2x4 array of ones
np.eye(3)               # 3x3 identity matrix
```

### Array Attributes

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])
arr.ndim    # Number of dimensions: 2
arr.shape   # Dimensions: (2, 3)
arr.size    # Total elements: 6
arr.dtype   # Data type: int64
```

### Indexing and Slicing

```python
arr = np.array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])

# Basic indexing
arr[3]          # Single element: 3
arr[2:5]        # Slice: [2, 3, 4]

# Advanced indexing
arr[arr > 5]    # Boolean indexing
arr[[0, 3, 6]]  # Fancy indexing
```

## Implementation Patterns

### Pattern 1: Vectorization

```python
# Broadcasting operations
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

# Element-wise operations
c = a + b        # [5, 7, 9]
d = a * 10       # [10, 20, 30]
e = np.sin(a)    # [sin(1), sin(2), sin(3)]
```

### Pattern 2: Array Aggregation

```python
data = np.array([[1, 2, 3], [4, 5, 6]])

# Overall statistics
data.sum()       # Total sum
data.mean()      # Mean value
data.max()       # Maximum value

# Axis-specific operations
data.sum(axis=0) # Column sums
data.mean(axis=1)# Row means
```

## Common Challenges and Solutions

### Challenge 1: Broadcasting Mismatch

**Solution:**
```python
# Ensure compatible shapes for broadcasting
a = np.array([[1, 2, 3]])  # Shape (1, 3)
b = np.array([[4], [5]])   # Shape (2, 1)
result = a + b             # Broadcasting works to (2, 3)
```

### Challenge 2: Array Copy vs View

**Solution:**
```python
# Create actual copy when needed
original = np.array([1, 2, 3, 4])
view = original[1:3]          # View, not copy
copy = original[1:3].copy()   # Actual copy
```

## Practical Example

```python
# Data analysis with NumPy
data = np.random.randn(1000, 10)  # 1000 samples, 10 features

# Basic statistics
mean_features = data.mean(axis=0)
std_features = data.std(axis=0)

# Standardization
standardized = (data - mean_features) / std_features

# Correlation matrix
correlation = np.corrcoef(data, rowvar=False)

# Finding outliers
threshold = 3
outliers = np.abs(data) > threshold
num_outliers = outliers.sum(axis=0)
```

## Summary

NumPy is essential for:
1. Efficient numerical computation
2. Array manipulation and transformation
3. Foundation for other scientific libraries
4. Fast vectorized operations

Remember that NumPy operations are typically much faster than pure Python loops for numerical tasks.

## Next Steps

Now that you understand NumPy fundamentals, the next step is learning Pandas for data manipulation with labeled data structures.

---

[<- Back to Main Topic](./13-data-science.md) | [Next Sub-Topic: Pandas Data Manipulation ->](./13b-pandas-data-manipulation.md)

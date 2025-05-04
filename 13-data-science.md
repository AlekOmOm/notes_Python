# 13. Data Science and Analysis Tools 📊

[<- Back: Testing and Documentation](./12-testing.md) | [Next: (Consider adding a project or further steps) ->]

## Table of Contents

- [Introduction to Data Science Libraries](#introduction-to-data-science-libraries)
- [13a. NumPy: Numerical Python](./13a-numpy-fundamentals.md)
- [13b. Pandas: Data Manipulation and Analysis](./13b-pandas-data-manipulation.md)
- [13c. Matplotlib: Data Visualization](./13c-matplotlib-visualization.md)

## Introduction to Data Science Libraries

Python has become a dominant language in data science due to its readability, extensive libraries, and strong community support. Three libraries form the foundation of many data science workflows:

1.  **NumPy:** Provides efficient multi-dimensional array objects and tools for numerical computation.
2.  **Pandas:** Offers high-performance, easy-to-use data structures (like DataFrames) and data analysis tools.
3.  **Matplotlib:** A comprehensive library for creating static, animated, and interactive visualizations.

## Installation

You'll typically install these libraries using pip:

```bash
pip install numpy pandas matplotlib
```

## Learning Path

This section is divided into three sub-topics to cover each library in detail:

1. **[NumPy Fundamentals](./13a-numpy-fundamentals.md)** - Learn array manipulation and numerical operations
2. **[Pandas Data Manipulation](./13b-pandas-data-manipulation.md)** - Master data structures and data analysis
3. **[Matplotlib Visualization](./13c-matplotlib-visualization.md)** - Create compelling data visualizations

Each sub-topic includes:
- Core concepts and terminology
- Practical code examples
- Common patterns and best practices
- Real-world applications

## Quick Reference

### NumPy
```python
import numpy as np
arr = np.array([1, 2, 3, 4, 5])
```

### Pandas
```python
import pandas as pd
df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})
```

### Matplotlib
```python
import matplotlib.pyplot as plt
plt.plot([1, 2, 3], [4, 5, 6])
plt.show()
```

## Summary

These three libraries work together to form the backbone of Python's data science ecosystem. Understanding them well is crucial for effective data analysis and visualization.

---

[<- Back: Testing and Documentation](./12-testing.md) | [Next: (Consider adding a project or further steps) ->]

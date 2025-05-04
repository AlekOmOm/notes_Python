# 13b. Pandas: Data Manipulation and Analysis 🐼

[<- Back to Main Topic](./13-data-science.md) | [Next Sub-Topic: Matplotlib Visualization ->](./13c-matplotlib-visualization.md)

## Overview

Pandas provides high-performance, easy-to-use data structures and data analysis tools built on top of NumPy. It's designed for working with "relational" or "labeled" data (like tables in SQL or spreadsheets).

Key features:
* **DataFrame object:** A 2D labeled data structure with columns of potentially different types.
* **Series object:** A 1D labeled array.
* Tools for reading and writing data between in-memory data structures and different file formats.
* Intelligent data alignment and integrated handling of missing data.

## Key Concepts

### Import Convention

```python
import pandas as pd
```

### Core Data Structures

```python
# Series (1D labeled array)
s = pd.Series([1, 2, 3, 4], index=['a', 'b', 'c', 'd'])

# DataFrame (2D labeled data structure)
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'city': ['New York', 'Paris', 'London']
})
```

### Reading/Writing Data

```python
# Read CSV
df = pd.read_csv('data.csv')

# Read Excel
df = pd.read_excel('data.xlsx', sheet_name='Sheet1')

# Write to files
df.to_csv('output.csv', index=False)
df.to_excel('output.xlsx', index=False)
```

## Implementation Patterns

### Pattern 1: Data Selection

```python
# Column selection
df['age']               # Single column
df[['name', 'age']]     # Multiple columns

# Row selection
df.loc['row_label']     # By label
df.iloc[0]              # By position

# Combined selection
df.loc[0:2, ['name', 'age']]    # First 3 rows, selected columns
```

### Pattern 2: Data Transformation

```python
# Adding/modifying columns
df['salary'] = [50000, 60000, 75000]
df['age_group'] = df['age'].apply(lambda x: 'Young' if x < 30 else 'Middle')

# Dropping columns/rows
df.drop('age_group', axis=1)    # Drop column
df.drop(0, axis=0)              # Drop row

# Sorting
df.sort_values('age', ascending=False)
df.sort_values(['city', 'age'], ascending=[True, False])
```

### Pattern 3: Data Aggregation

```python
# GroupBy operations
grouped = df.groupby('city')
grouped['age'].mean()           # Average age per city
grouped.agg({
    'age': ['mean', 'max'],
    'salary': ['sum', 'min']
})

# Summary statistics
df.describe()                   # Numerical columns
df.info()                       # DataFrame info
```

## Common Challenges and Solutions

### Challenge 1: Missing Data

**Solution:**
```python
# Handle missing values
df.isnull().sum()              # Count missing values
df.dropna()                    # Drop rows with any NaN
df.fillna(0)                   # Fill NaN with 0
df['age'].fillna(df['age'].mean()) # Fill with column mean
```

### Challenge 2: Data Merging

**Solution:**
```python
# Merge DataFrames
left_df = pd.DataFrame({'key': ['A', 'B'], 'value': [1, 2]})
right_df = pd.DataFrame({'key': ['A', 'C'], 'value': [3, 4]})

# Different join types
pd.merge(left_df, right_df, on='key', how='inner')
pd.merge(left_df, right_df, on='key', how='left')
pd.merge(left_df, right_df, on='key', how='outer')
```

## Practical Example

```python
# Real-world data analysis example
import pandas as pd

# Load sales data
sales_df = pd.read_csv('sales_data.csv')

# Clean data
sales_df['date'] = pd.to_datetime(sales_df['date'])
sales_df = sales_df.dropna(subset=['amount'])

# Calculate metrics
sales_df['month'] = sales_df['date'].dt.month
monthly_sales = sales_df.groupby('month')['amount'].sum()

# Find best performing products
top_products = sales_df.groupby('product')['amount'].sum().nlargest(10)

# Create summary report
summary = {
    'total_sales': sales_df['amount'].sum(),
    'average_order': sales_df['amount'].mean(),
    'top_month': monthly_sales.idxmax(),
    'best_product': top_products.index[0]
}
print(summary)
```

## Summary

Pandas excels at:
1. Data cleaning and preparation
2. Data exploration and analysis
3. Time series manipulation
4. Data aggregation and grouping
5. Handling missing data

Its DataFrame structure makes it ideal for working with structured, tabular data.

## Next Steps

Once comfortable with Pandas for data manipulation, the next natural step is learning Matplotlib for data visualization.

---

[<- Back to Main Topic](./13-data-science.md) | [Next Sub-Topic: Matplotlib Visualization ->](./13c-matplotlib-visualization.md)

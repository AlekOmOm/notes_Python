# 13c. Matplotlib: Data Visualization 📈

[<- Back to Main Topic](./13-data-science.md) | [Next Sub-Topic: (End of section) ->](./13-data-science.md)

## Overview

Matplotlib is a popular and versatile Python library for creating static, animated, and interactive visualizations. It provides fine-grained control over plots and is often used with NumPy and Pandas.

Key features:
* MATLAB-like interface for quick plotting
* Object-oriented interface for custom visualizations
* Wide variety of plot types
* Extensive customization options
* Integration with other scientific libraries

## Key Concepts

### Import Convention

```python
import matplotlib.pyplot as plt
```

### Basic Plot Anatomy

```python
# Figure and Axes concept
fig, ax = plt.subplots()

# Plot data
ax.plot(x, y)

# Customize
ax.set_title("My Plot")
ax.set_xlabel("X-axis")
ax.set_ylabel("Y-axis")

# Display
plt.show()
```

### Common Plot Types

```python
# Line plot
plt.plot(x, y)

# Scatter plot
plt.scatter(x, y, s=sizes, c=colors, alpha=0.6)

# Bar plot
plt.bar(categories, values)

# Histogram
plt.hist(data, bins=30)

# Box plot
plt.boxplot([data1, data2, data3])
```

## Implementation Patterns

### Pattern 1: Figure and Axes Management

```python
# Single plot
fig, ax = plt.subplots()
ax.plot(x, y)

# Subplots grid
fig, axes = plt.subplots(2, 2, figsize=(10, 8))
axes[0, 0].plot(x, y1)
axes[0, 1].scatter(x, y2)
axes[1, 0].hist(data1)
axes[1, 1].bar(categories, values)

# Layout adjustment
plt.tight_layout()
```

### Pattern 2: Style Customization

```python
# Colors and styles
ax.plot(x, y, color='red', linestyle='--', linewidth=2, marker='o')

# Multiple series with legend
ax.plot(x, y1, label='Series 1')
ax.plot(x, y2, label='Series 2')
ax.legend()

# Grid and limits
ax.grid(True, alpha=0.3)
ax.set_xlim(0, 10)
ax.set_ylim(-1, 1)
```

### Pattern 3: Integration with Pandas

```python
# Direct plotting from DataFrame
df.plot(kind='bar', x='category', y='value', ax=ax)
df.plot(kind='line', ax=ax)
df.plot(kind='scatter', x='age', y='salary', ax=ax)

# Group by and plot
grouped_data = df.groupby('category')['value'].mean()
grouped_data.plot(kind='bar', ax=ax)
```

## Common Challenges and Solutions

### Challenge 1: Overlapping Labels

**Solution:**
```python
# Rotate labels
plt.xticks(rotation=45)

# Adjust layout
plt.tight_layout()

# Adjust figure size
fig, ax = plt.subplots(figsize=(12, 6))
```

### Challenge 2: Plot Customization

**Solution:**
```python
# Complete customization
fig, ax = plt.subplots()
ax.plot(x, y, color='navy', linewidth=2)
ax.set_title('Custom Plot', fontsize=16, fontweight='bold')
ax.set_xlabel('X-axis Label', fontsize=12)
ax.set_ylabel('Y-axis Label', fontsize=12)
ax.grid(True, linestyle='--', alpha=0.7)
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
```

## Practical Example

```python
# Comprehensive data visualization example
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

# Generate sample data
dates = pd.date_range('2023-01-01', periods=100)
df = pd.DataFrame({
    'date': dates,
    'sales': np.random.randint(100, 1000, 100),
    'costs': np.random.randint(50, 500, 100),
    'profit': np.random.randint(-100, 500, 100)
})

# Create visualization dashboard
fig = plt.figure(figsize=(15, 10))
gs = fig.add_gridspec(3, 2, height_ratios=[2, 1, 1])

# Main time series plot
ax1 = fig.add_subplot(gs[0, :])
ax1.plot(df['date'], df['sales'], label='Sales', linewidth=2)
ax1.plot(df['date'], df['costs'], label='Costs', linewidth=2)
ax1.fill_between(df['date'], df['sales'], df['costs'], 
                 where=(df['sales'] > df['costs']), 
                 alpha=0.3, color='green', label='Profit')
ax1.set_title('Sales and Costs Over Time', fontsize=16)
ax1.set_xlabel('Date')
ax1.set_ylabel('Amount ($)')
ax1.legend()
ax1.grid(True, alpha=0.3)

# Profit distribution
ax2 = fig.add_subplot(gs[1, 0])
ax2.hist(df['profit'], bins=20, edgecolor='black', alpha=0.7)
ax2.set_title('Profit Distribution')
ax2.set_xlabel('Profit ($)')
ax2.set_ylabel('Frequency')

# Monthly aggregation
monthly_sales = df.groupby(df['date'].dt.month)['sales'].mean()
ax3 = fig.add_subplot(gs[1, 1])
ax3.bar(monthly_sales.index, monthly_sales.values, color='skyblue', edgecolor='black')
ax3.set_title('Average Monthly Sales')
ax3.set_xlabel('Month')
ax3.set_ylabel('Average Sales ($)')

# Scatter plot
ax4 = fig.add_subplot(gs[2, :])
scatter = ax4.scatter(df['sales'], df['costs'], 
                      c=df['profit'], cmap='RdYlGn', 
                      alpha=0.6, s=100)
ax4.set_title('Sales vs Costs (colored by Profit)')
ax4.set_xlabel('Sales ($)')
ax4.set_ylabel('Costs ($)')
plt.colorbar(scatter, label='Profit ($)')

plt.tight_layout()
plt.savefig('sales_dashboard.png', dpi=300, bbox_inches='tight')
plt.show()
```

## Summary

Matplotlib is powerful for:
1. Creating publication-quality figures
2. Customizing every aspect of plots
3. Integrating with scientific workflows
4. Building complex visualizations
5. Supporting various plot types

Remember that while Matplotlib is flexible, sometimes higher-level libraries like Seaborn can simplify common visualization tasks.

## Next Steps

Having completed all three sub-topics, you now have a solid foundation in Python's data science ecosystem. Consider exploring:
- Seaborn for statistical visualizations
- Scikit-learn for machine learning
- Plotly for interactive visualizations

---

[<- Back to Main Topic](./13-data-science.md) | [Next Sub-Topic: (End of section) ->](./13-data-science.md)

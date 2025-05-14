# 17b. Data Processing and Visualization 📊

[<- Back to Main Topic](./17-flask-socketio.md) | [Previous Sub-Topic: Core Application Structure](./17a-core-application.md)

## Overview

This sub-topic explores the data handling and visualization aspects of real-time dashboards using Flask-SocketIO. We'll cover how to process data with pandas, create interactive visualizations with Plotly, and efficiently stream updates to clients. These techniques enable building responsive, data-rich dashboards that display real-time information without requiring page refreshes.

## Key Concepts

### Data Flow Architecture

```
Data Source → Pandas Processing → JSON Serialization → Socket Emission → Client Rendering
```

The data pipeline follows these steps:
1. **Data Acquisition**: Reading from files, databases, APIs, or streams
2. **Transformation**: Processing with pandas to prepare for visualization
3. **Serialization**: Converting to a format suitable for transmission
4. **Transmission**: Sending to clients via WebSockets
5. **Rendering**: Displaying in the browser with Plotly

### Optimization Considerations

- **Memory Management**: Caching strategies and dataframe size control
- **Transmission Efficiency**: Sending deltas vs. complete datasets
- **Rendering Performance**: Client-side visualization optimization
- **Update Frequency**: Balancing real-time needs vs. system load

## Implementation Patterns

### Pattern 1: Data Processing Service

```python
# app/services.py
import pandas as pd
import numpy as np
import plotly.express as px
import plotly.graph_objects as go
from datetime import datetime, timedelta
from pathlib import Path
import json

# Cache for in-memory data storage
_data_cache = None
_last_load_time = None

def current_frame(force_reload=False, max_age_seconds=60):
    """
    Get the current DataFrame, reloading if necessary.
    
    Args:
        force_reload: Force a reload regardless of cache state
        max_age_seconds: Maximum age of cache before automatic reload
    
    Returns:
        pandas.DataFrame: The current dataset
    """
    global _data_cache, _last_load_time
    now = datetime.now()
    
    # Determine if reload is needed
    cache_expired = (_last_load_time is None or 
                     (now - _last_load_time).total_seconds() > max_age_seconds)
    
    if _data_cache is None or force_reload or cache_expired:
        # In a real application, this might load from a database
        # or an API rather than a static file
        data_path = Path("data/metrics.csv")
        _data_cache = pd.read_csv(data_path, parse_dates=["timestamp"])
        _last_load_time = now
        
    return _data_cache

def apply_filters(df, filters=None):
    """Apply filters to the DataFrame."""
    if filters is None:
        return df
        
    filtered_df = df.copy()
    
    # Apply each filter
    if "time_range" in filters:
        hours = filters["time_range"]
        cutoff = datetime.now() - timedelta(hours=hours)
        filtered_df = filtered_df[filtered_df["timestamp"] >= cutoff]
        
    if "category" in filters and filters["category"]:
        filtered_df = filtered_df[filtered_df["category"] == filters["category"]]
        
    return filtered_df

def aggregate_data(df, groupby="hour", agg_func="mean"):
    """
    Aggregate data at the specified granularity.
    
    Args:
        df: Input DataFrame
        groupby: Granularity for grouping ('minute', 'hour', 'day')
        agg_func: Aggregation function ('mean', 'sum', 'max', etc)
        
    Returns:
        Aggregated DataFrame
    """
    if groupby == "minute":
        time_col = df["timestamp"].dt.floor("min")
    elif groupby == "hour":
        time_col = df["timestamp"].dt.floor("H")
    elif groupby == "day":
        time_col = df["timestamp"].dt.floor("D")
    else:
        # Default to hourly
        time_col = df["timestamp"].dt.floor("H")
        
    grouped = df.groupby([time_col]).agg({
        "value": agg_func
    }).reset_index()
    
    return grouped
```

**When to use this pattern:**
- When processing or aggregating data before visualization
- When working with data from multiple sources
- When implementing caching strategies for performance

### Pattern 2: Visualization Generation

```python
def build_line_chart(df, title="Time Series Data"):
    """
    Create a line chart visualization from time series data.
    
    Args:
        df: pandas DataFrame with 'timestamp' and 'value' columns
        title: Chart title
        
    Returns:
        JSON-serialized Plotly figure
    """
    # Use plotly express for simple charts
    fig = px.line(
        df, 
        x="timestamp", 
        y="value", 
        title=title,
        labels={"timestamp": "Time", "value": "Value"}
    )
    
    # Customize layout
    fig.update_layout(
        margin=dict(l=40, r=40, t=40, b=40),
        hovermode="closest",
        plot_bgcolor="white",
        xaxis=dict(
            showgrid=True,
            gridcolor="lightgray"
        ),
        yaxis=dict(
            showgrid=True,
            gridcolor="lightgray"
        )
    )
    
    # Convert to JSON for socket transmission
    return fig.to_json()

def build_multi_series_chart(df, category_col="category", title="Multi-Series Data"):
    """
    Create a multi-series line chart with data grouped by category.
    
    Args:
        df: pandas DataFrame with 'timestamp', 'value', and category columns
        category_col: Column name to use for grouping
        title: Chart title
        
    Returns:
        JSON-serialized Plotly figure
    """
    fig = px.line(
        df,
        x="timestamp", 
        y="value",
        color=category_col,
        title=title,
        labels={"timestamp": "Time", "value": "Value", category_col: "Category"}
    )
    
    fig.update_layout(
        legend=dict(
            orientation="h",
            yanchor="bottom",
            y=1.02,
            xanchor="right",
            x=1
        )
    )
    
    return fig.to_json()

def build_dashboard(df, filters=None):
    """
    Create a complete dashboard with multiple visualization components.
    
    Args:
        df: Source DataFrame
        filters: Optional filters to apply
        
    Returns:
        Dictionary of JSON-serialized Plotly figures
    """
    # Apply filters if provided
    filtered_df = apply_filters(df, filters)
    
    # Get the latest value for KPI display
    latest_value = filtered_df["value"].iloc[-1] if not filtered_df.empty else 0
    
    # Calculate change from previous period
    if len(filtered_df) > 1:
        previous_value = filtered_df["value"].iloc[-2]
        percent_change = ((latest_value - previous_value) / previous_value) * 100
    else:
        percent_change = 0
    
    # Generate visualizations
    time_series = build_line_chart(
        filtered_df.tail(200),  # Limit points for performance
        title="Recent Metrics"
    )
    
    # Create hourly aggregation
    hourly_data = aggregate_data(filtered_df, groupby="hour")
    hourly_chart = build_line_chart(
        hourly_data,
        title="Hourly Average"
    )
    
    # Return a dictionary of visualizations
    return {
        "time_series": time_series,
        "hourly_chart": hourly_chart,
        "kpi": {
            "latest": float(latest_value),
            "percent_change": float(percent_change)
        }
    }
```

**When to use this pattern:**
- When building complex dashboards with multiple visualizations
- When you need consistent styling across charts
- When supporting dynamic filtering and aggregation

### Pattern 3: Efficient Data Transmission

```python
# app/sockets.py
from .services import current_frame, build_dashboard
import json

def register_events(sio, app):
    """Register Socket.IO event handlers."""
    
    # Track client-specific states
    client_filters = {}
    
    @sio.event
    def connect(sid, environ):
        """Handle new client connections."""
        app.logger.info({"event": "connect", "sid": sid})
        
        # Initialize client state
        client_filters[sid] = {}
        
        # Send initial dashboard data
        df = current_frame()
        dashboard = build_dashboard(df)
        sio.emit("update_dashboard", dashboard, to=sid)
    
    @sio.event
    def disconnect(sid):
        """Handle client disconnections."""
        app.logger.info({"event": "disconnect", "sid": sid})
        
        # Clean up client state
        if sid in client_filters:
            del client_filters[sid]
    
    @sio.event
    def apply_filters(sid, data):
        """Apply filters to the dashboard."""
        app.logger.info({"event": "apply_filters", "sid": sid, "filters": data})
        
        # Store client's filter preferences
        client_filters[sid] = data
        
        # Generate filtered dashboard
        df = current_frame()
        dashboard = build_dashboard(df, filters=data)
        sio.emit("update_dashboard", dashboard, to=sid)
    
    @sio.event
    def request_data_subset(sid, data):
        """Fetch a specific subset of data."""
        chart_type = data.get("chart_type", "time_series")
        granularity = data.get("granularity", "hour")
        
        df = current_frame()
        
        # Apply client's stored filters
        filters = client_filters.get(sid, {})
        filtered_df = apply_filters(df, filters)
        
        # Aggregate data if requested
        if granularity != "raw":
            filtered_df = aggregate_data(filtered_df, groupby=granularity)
        
        # Build and send only the requested chart
        if chart_type == "time_series":
            chart_json = build_line_chart(filtered_df)
            sio.emit("update_chart", {"id": chart_type, "data": chart_json}, to=sid)
        elif chart_type == "hourly":
            hourly_data = aggregate_data(filtered_df, groupby="hour")
            chart_json = build_line_chart(hourly_data, "Hourly Average")
            sio.emit("update_chart", {"id": chart_type, "data": chart_json}, to=sid)
```

**When to use this pattern:**
- When optimizing for network efficiency with large datasets
- When supporting personalized views for different users
- When implementing interactive filtering and drill-down features

## Common Challenges and Solutions

### Challenge 1: Large Dataset Performance

**Solution:**
```python
def optimize_dataframe(df, max_points=1000):
    """Optimize a DataFrame for transmission by sampling if too large."""
    if len(df) <= max_points:
        return df
    
    # Option 1: Simple sampling
    # return df.sample(max_points)
    
    # Option 2: Systematic sampling preserving time series pattern
    sample_rate = len(df) // max_points
    return df.iloc[::sample_rate].copy()
    
    # Option 3: Temporal bucketing (better for time series)
    # grouped = df.groupby(pd.Grouper(key='timestamp', freq='5min')).mean()
    # return grouped.reset_index()
```

This solution addresses the common challenge of transmitting large datasets by:
1. Determining if optimization is needed
2. Using intelligent sampling strategies
3. Preserving important patterns in the data

### Challenge 2: Real-time Updates from External Sources

**Solution:**
```python
import threading
import time
from queue import Queue

update_queue = Queue()

def background_data_fetcher(interval=30):
    """
    Background thread that periodically checks for new data
    and pushes updates to connected clients.
    """
    while True:
        try:
            # Fetch latest data (from API, database, etc.)
            new_data = fetch_latest_data()
            
            # If new data is available, add to queue
            if new_data is not None:
                update_queue.put(new_data)
            
            # Sleep for the specified interval
            time.sleep(interval)
        except Exception as e:
            print(f"Error in background fetcher: {e}")
            time.sleep(interval)

def register_background_tasks(sio, app):
    """Register background tasks with the Flask application."""
    
    # Start background data fetcher
    fetcher_thread = threading.Thread(
        target=background_data_fetcher,
        daemon=True
    )
    fetcher_thread.start()
    
    # Process to check for updates and emit to clients
    def process_updates():
        while True:
            try:
                if not update_queue.empty():
                    new_data = update_queue.get()
                    
                    # Update the in-memory cache
                    update_data_cache(new_data)
                    
                    # Get the updated dashboard data
                    df = current_frame()
                    dashboard = build_dashboard(df)
                    
                    # Broadcast to all connected clients
                    sio.emit("update_dashboard", dashboard)
                
                # Check every second
                sio.sleep(1)
            except Exception as e:
                app.logger.error({"event": "update_error", "error": str(e)})
    
    # Start the update processor using socketio's background task feature
    sio.start_background_task(process_updates)
```

This pattern sets up a robust system for handling external data updates by:
1. Running a background thread to fetch new data
2. Using a thread-safe queue to communicate between threads
3. Processing updates in a Socket.IO background task
4. Broadcasting changes to all connected clients

## Practical Example: Complete Dashboard Implementation

Here's a complete example integrating the data processing and visualization components:

### Server-Side Implementation

```python
# app/services.py
import pandas as pd
import plotly.express as px
from datetime import datetime, timedelta
from pathlib import Path

# Cache for data
_data_cache = None
_last_load_time = None

def current_frame(force_reload=False):
    """Get current data, refreshing if needed."""
    global _data_cache, _last_load_time
    now = datetime.now()
    
    if (_data_cache is None or force_reload or 
            (_last_load_time and (now - _last_load_time).total_seconds() > 60)):
        data_path = Path("data/metrics.csv")
        _data_cache = pd.read_csv(data_path, parse_dates=["timestamp"])
        _last_load_time = now
    
    return _data_cache

def filter_by_timeframe(df, hours=24):
    """Filter to the last N hours of data."""
    cutoff = datetime.now() - timedelta(hours=hours)
    return df[df["timestamp"] >= cutoff]

def build_dashboard(df, time_window=24):
    """Build complete dashboard with multiple visualizations."""
    # Filter data to requested time window
    recent_data = filter_by_timeframe(df, hours=time_window)
    
    # Time series chart
    time_chart = px.line(
        recent_data.tail(500),
        x="timestamp",
        y="value",
        title=f"Metrics - Last {time_window} Hours"
    )
    time_chart.update_layout(
        margin=dict(l=40, r=40, t=60, b=40),
        xaxis_title="Time",
        yaxis_title="Value"
    )
    
    # Summary statistics
    stats = {
        "latest": float(recent_data["value"].iloc[-1]) if not recent_data.empty else 0,
        "mean": float(recent_data["value"].mean()),
        "min": float(recent_data["value"].min()),
        "max": float(recent_data["value"].max())
    }
    
    # Hourly aggregation
    hourly = recent_data.groupby(recent_data["timestamp"].dt.floor("H")).mean().reset_index()
    hourly_chart = px.bar(
        hourly,
        x="timestamp",
        y="value",
        title="Hourly Averages"
    )
    
    # Dashboard package
    return {
        "time_series": time_chart.to_json(),
        "hourly": hourly_chart.to_json(),
        "stats": stats
    }

# app/sockets.py
def register_events(sio, app):
    """Register Socket.IO event handlers."""
    
    # Client state tracking
    client_settings = {}
    
    @sio.event
    def connect(sid, environ):
        """Handle new client connection."""
        app.logger.info({"event": "connect", "sid": sid})
        
        # Initialize default settings
        client_settings[sid] = {
            "time_window": 24,
            "refresh_interval": 30
        }
        
        # Send initial dashboard
        df = current_frame()
        dashboard = build_dashboard(df)
        sio.emit("update_dashboard", dashboard, to=sid)
        
        # Schedule periodic updates
        def send_updates():
            while True:
                # Get client-specific settings
                if sid not in client_settings:
                    break  # Client disconnected
                    
                time_window = client_settings[sid]["time_window"]
                interval = client_settings[sid]["refresh_interval"]
                
                # Update dashboard
                try:
                    df = current_frame(force_reload=True)
                    dashboard = build_dashboard(df, time_window=time_window)
                    sio.emit("update_dashboard", dashboard, to=sid)
                except Exception as e:
                    app.logger.error({"event": "update_error", "sid": sid, "error": str(e)})
                
                # Wait for next update
                sio.sleep(interval)
        
        # Start background task for this client
        sio.start_background_task(send_updates)
    
    @sio.event
    def disconnect(sid):
        """Handle client disconnection."""
        app.logger.info({"event": "disconnect", "sid": sid})
        
        # Clean up client state
        if sid in client_settings:
            del client_settings[sid]
    
    @sio.event
    def update_settings(sid, settings):
        """Update client dashboard settings."""
        app.logger.info({"event": "update_settings", "sid": sid, "settings": settings})
        
        # Update stored settings
        if sid in client_settings:
            client_settings[sid].update(settings)
            
            # Send immediate update with new settings
            time_window = client_settings[sid]["time_window"]
            df = current_frame()
            dashboard = build_dashboard(df, time_window=time_window)
            sio.emit("update_dashboard", dashboard, to=sid)
```

### Client-Side Implementation

```html
<!-- app/templates/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Real-time Analytics Dashboard</title>
    <!-- Socket.IO and Plotly libraries -->
    <script src="https://cdn.socket.io/4.7.4/socket.io.min.js"></script>
    <script src="https://cdn.plot.ly/plotly-2.29.1.min.js"></script>
    <link rel="stylesheet" href="{{ url_for('static', filename='dash.css') }}">
</head>
<body>
    <header>
        <h1>Real-time Analytics Dashboard</h1>
        <div class="controls">
            <label for="time-window">Time Window:</label>
            <select id="time-window">
                <option value="4">Last 4 Hours</option>
                <option value="24" selected>Last 24 Hours</option>
                <option value="72">Last 3 Days</option>
                <option value="168">Last Week</option>
            </select>
            
            <label for="refresh-rate">Refresh Rate:</label>
            <select id="refresh-rate">
                <option value="10">10 seconds</option>
                <option value="30" selected>30 seconds</option>
                <option value="60">1 minute</option>
                <option value="300">5 minutes</option>
            </select>
            
            <button id="refresh-now">Refresh Now</button>
        </div>
    </header>
    
    <div class="dashboard">
        <div class="stats-panel">
            <div class="stat-card" id="latest-value">
                <div class="stat-title">Current Value</div>
                <div class="stat-value">0</div>
            </div>
            <div class="stat-card" id="avg-value">
                <div class="stat-title">Average</div>
                <div class="stat-value">0</div>
            </div>
            <div class="stat-card" id="min-value">
                <div class="stat-title">Minimum</div>
                <div class="stat-value">0</div>
            </div>
            <div class="stat-card" id="max-value">
                <div class="stat-title">Maximum</div>
                <div class="stat-value">0</div>
            </div>
        </div>
        
        <div class="chart-container">
            <div id="time-series-chart" class="chart"></div>
            <div id="hourly-chart" class="chart"></div>
        </div>
    </div>
    
    <script src="{{ url_for('static', filename='dash.js') }}"></script>
</body>
</html>
```

```javascript
// app/static/dash.js
// Initialize Socket.IO connection
const socket = io();

// Dashboard state
let dashboardData = {};

// DOM elements
const timeWindowSelect = document.getElementById('time-window');
const refreshRateSelect = document.getElementById('refresh-rate');
const refreshButton = document.getElementById('refresh-now');

// Initialize charts
const timeSeriesChart = document.getElementById('time-series-chart');
const hourlyChart = document.getElementById('hourly-chart');

// Stats elements
const latestValue = document.getElementById('latest-value').querySelector('.stat-value');
const avgValue = document.getElementById('avg-value').querySelector('.stat-value');
const minValue = document.getElementById('min-value').querySelector('.stat-value');
const maxValue = document.getElementById('max-value').querySelector('.stat-value');

// Connect to Socket.IO server
socket.on('connect', () => {
    console.log('Connected to server');
});

// Handle dashboard updates
socket.on('update_dashboard', (data) => {
    dashboardData = data;
    updateDashboard();
});

// Update UI with dashboard data
function updateDashboard() {
    // Update time series chart
    if (dashboardData.time_series) {
        const timeSeriesData = JSON.parse(dashboardData.time_series);
        Plotly.react(timeSeriesChart, timeSeriesData.data, timeSeriesData.layout);
    }
    
    // Update hourly chart
    if (dashboardData.hourly) {
        const hourlyData = JSON.parse(dashboardData.hourly);
        Plotly.react(hourlyChart, hourlyData.data, hourlyData.layout);
    }
    
    // Update statistics
    if (dashboardData.stats) {
        latestValue.textContent = dashboardData.stats.latest.toFixed(2);
        avgValue.textContent = dashboardData.stats.mean.toFixed(2);
        minValue.textContent = dashboardData.stats.min.toFixed(2);
        maxValue.textContent = dashboardData.stats.max.toFixed(2);
    }
}

// Event handlers for controls
timeWindowSelect.addEventListener('change', () => {
    const timeWindow = parseInt(timeWindowSelect.value);
    socket.emit('update_settings', { time_window: timeWindow });
});

refreshRateSelect.addEventListener('change', () => {
    const refreshInterval = parseInt(refreshRateSelect.value);
    socket.emit('update_settings', { refresh_interval: refreshInterval });
});

refreshButton.addEventListener('click', () => {
    socket.emit('update_settings', { 
        time_window: parseInt(timeWindowSelect.value),
        refresh_interval: parseInt(refreshRateSelect.value)
    });
});
```

## Summary

Key takeaways from this data processing and visualization section:
1. Pandas provides powerful data manipulation capabilities for preparing visualization data
2. Plotly generates interactive visualizations that can be serialized for WebSocket transmission
3. Client-specific state management enables personalized dashboard experiences
4. Data optimization strategies are essential for handling large datasets
5. Background processing enables continuous updates without blocking the main application
6. Structured dashboard organization helps manage complex multi-visualization interfaces

## Next Steps

After mastering these data processing and visualization techniques, consider exploring:
- Advanced visualization libraries like D3.js for custom visualizations
- Integration with ML libraries for predictive analytics
- Stream processing frameworks like Apache Kafka for high-volume data ingestion
- Time-series databases for efficient storage and querying of historical data
- Microservice architectures for scaling complex dashboard systems

---

[<- Back to Main Topic](./17-flask-socketio.md) | [Previous Sub-Topic: Core Application Structure](./17a-core-application.md)

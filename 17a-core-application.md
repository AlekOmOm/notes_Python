# 17a. Core Application Structure 🏗️

[<- Back to Main Topic](./17-flask-socketio.md) | [Next Sub-Topic: Data Processing and Visualization ->](./17b-data-processing.md)

## Overview

This sub-topic covers the essential architecture and implementation of a real-time Flask-SocketIO application. We'll build a foundation for a real-time analytics dashboard that streams live data updates to connected clients using a combination of Flask for HTTP handling and SocketIO for bidirectional communication.

## Key Concepts

### Architecture Flow

```
Client (Browser/JS) ─── HTTP GET → Flask → HTML+JS
Client ─── WebSocket "subscribe" → Flask-SocketIO
Flask  ─ Read => Data Source → Pandas → DataFrame
       ─ Process => Data Transformations 
       ─ Emit Chart JSON → Socket "update_chart"
Client JS Plotly.newPlot() ← Chart JSON
```

The architecture follows these principles:
- **Single Source of Truth**: Data maintained in server-side pandas DataFrame
- **Reactive Push Model**: Changes to data are pushed to clients automatically
- **Separation of Concerns**: HTTP serves the application shell, SocketIO handles data streaming, client-side libraries handle visualization

### Project Structure

```
realtime_dash/
│  config.py            # Configuration settings
│  wsgi.py              # Production entry point
└─ app/
   ├─ __init__.py       # Application factory
   ├─ routes.py         # HTTP endpoints
   ├─ sockets.py        # SocketIO event handlers
   ├─ services.py       # Data processing logic
   ├─ static/
   │   └─ dash.js       # Client-side socket + visualization
   └─ templates/
      └─ index.html     # Main application template
```

This structure follows the patterns from the intermediate Flask module while adding specific components for real-time communication.

## Implementation Patterns

### Pattern 1: Application Factory with SocketIO

```python
# app/__init__.py
from flask import Flask
from flask_socketio import SocketIO
from pythonjsonlogger import jsonlogger
import logging

# Create SocketIO instance outside the app factory
socketio = SocketIO(cors_allowed_origins="*")

def create_app():
    app = Flask(__name__)
    app.config.from_object("config.Config")
    
    # Configure structured JSON logging
    handler = logging.StreamHandler()
    handler.setFormatter(jsonlogger.JsonFormatter())
    app.logger.addHandler(handler)
    
    # Register blueprints and socket handlers
    from .routes import bp as http_bp
    from .sockets import register_events
    app.register_blueprint(http_bp)
    
    # Initialize SocketIO with the Flask app
    socketio.init_app(app)
    register_events(socketio, app)
    
    return app
```

**When to use this pattern:**
- When building real-time applications requiring both HTTP and WebSocket protocols
- When you need a clean separation between HTTP routes and socket event handlers
- When structured logging will help with monitoring and debugging

### Pattern 2: HTTP and WebSocket Separation

```python
# app/routes.py - Handles HTTP requests
from flask import Blueprint, render_template

bp = Blueprint("http", __name__)

@bp.get("/")
def index():
    """Serve the main application template."""
    return render_template("index.html")

# app/sockets.py - Handles WebSocket events
from .services import current_frame, build_chart

def register_events(sio, app):
    @sio.event
    def connect(sid, environ):
        """Handle new client connections."""
        app.logger.info({"event": "connect", "sid": sid})
        # Send initial chart data to the new client
        sio.emit("update_chart", build_chart(current_frame()), to=sid)
    
    @sio.event
    def refresh_data(sid):
        """Handle client refresh requests."""
        df = current_frame(force_reload=True)
        sio.emit("update_chart", build_chart(df))
```

**When to use this pattern:**
- When you want clear separation between HTTP and WebSocket concerns
- When different team members might work on different aspects of the application
- When you need to scale different parts of the system independently

### Pattern 3: Data Service Layer

```python
# app/services.py
import pandas as pd
import plotly.express as px
from pathlib import Path

DATA = Path("data/metrics.csv")
_cache = None

def current_frame(force_reload=False):
    """Get the current DataFrame, reloading if necessary."""
    global _cache
    if _cache is None or force_reload:
        _cache = pd.read_csv(DATA, parse_dates=["timestamp"])
    return _cache

def build_chart(df: pd.DataFrame):
    """Create a JSON-serializable chart from DataFrame."""
    latest = df.tail(200)  # Limit to recent data points
    fig = px.line(latest, x="timestamp", y="value", title="Live Metric")
    return fig.to_json()  # Serializable for socket transmission
```

**When to use this pattern:**
- When you need a clear separation between data processing and communication
- When data transformations are complex and warrant their own module
- When you want to cache data in memory for performance

## Common Challenges and Solutions

### Challenge 1: Asynchronous Execution Models

**Solution:**
```python
# wsgi.py - Entry point with proper async worker configuration
from app import create_app, socketio

app = create_app()

if __name__ == "__main__":
    socketio.run(app, debug=True)
```

When deploying with Gunicorn:
```bash
gunicorn 'app:create_app()' --worker-class eventlet --workers 1
```

Flask-SocketIO requires special workers (eventlet or gevent) that support WebSockets and asynchronous I/O. Standard WSGI workers won't handle WebSocket connections properly.

### Challenge 2: Client Connection Management

**Solution:**
```python
# Advanced room-based broadcasting for grouped clients
@sio.event
def join_dashboard(sid, dashboard_id):
    """Add a client to a specific dashboard room."""
    sio.enter_room(sid, f"dashboard_{dashboard_id}")
    app.logger.info({
        "event": "join_dashboard", 
        "sid": sid, 
        "dashboard": dashboard_id
    })

def broadcast_update(dashboard_id, data):
    """Send updates to all clients viewing a specific dashboard."""
    sio.emit(
        "update_chart", 
        data, 
        room=f"dashboard_{dashboard_id}"
    )
```

Using rooms lets you organize connections logically and target emissions efficiently, especially when scaling to many simultaneous users.

## Practical Example: Complete Core Application

Here's a complete example of the core application structure:

```python
# config.py
class Config:
    """Application configuration."""
    SECRET_KEY = "dev-change-this-in-production"
    DEBUG = True
    JSON_SORT_KEYS = False
    
# wsgi.py
from app import create_app, socketio

app = create_app()

if __name__ == "__main__":
    socketio.run(app, debug=True)

# app/__init__.py
from flask import Flask
from flask_socketio import SocketIO
from pythonjsonlogger import jsonlogger
import logging

socketio = SocketIO(cors_allowed_origins="*")

def create_app():
    app = Flask(__name__)
    app.config.from_object("config.Config")
    
    # Configure structured logging
    handler = logging.StreamHandler()
    handler.setFormatter(jsonlogger.JsonFormatter())
    app.logger.handlers = [handler]
    app.logger.setLevel(logging.INFO)
    
    # Register blueprints and socket handlers
    from .routes import bp
    app.register_blueprint(bp)
    
    socketio.init_app(app)
    
    from .sockets import register_events
    register_events(socketio, app)
    
    return app

# app/routes.py
from flask import Blueprint, render_template

bp = Blueprint("http", __name__)

@bp.get("/")
def index():
    """Serve the main dashboard page."""
    return render_template("index.html")

# app/sockets.py
from .services import current_frame, build_chart

def register_events(sio, app):
    @sio.event
    def connect(sid, environ):
        """Handle new client connections."""
        app.logger.info({"event": "connect", "sid": sid})
        # Send initial data to new client
        sio.emit("update_chart", build_chart(current_frame()), to=sid)
    
    @sio.event
    def disconnect(sid):
        """Handle client disconnections."""
        app.logger.info({"event": "disconnect", "sid": sid})
    
    @sio.event
    def refresh_data(sid):
        """Handle manual refresh requests."""
        app.logger.info({"event": "refresh_request", "sid": sid})
        df = current_frame(force_reload=True)
        sio.emit("update_chart", build_chart(df))
```

### Client-Side Implementation

```html
<!-- app/templates/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Real-time Dashboard</title>
    <!-- Socket.IO and Plotly libraries -->
    <script src="https://cdn.socket.io/4.7.4/socket.io.min.js"></script>
    <script src="https://cdn.plot.ly/plotly-2.29.1.min.js"></script>
    <style>
        body { font-family: sans-serif; max-width: 1200px; margin: 0 auto; padding: 20px; }
        #chart { width: 100%; height: 500px; }
        .controls { margin: 20px 0; }
    </style>
</head>
<body>
    <h1>Real-time Analytics Dashboard</h1>
    <div id="chart"></div>
    <div class="controls">
        <button id="refresh">Force Refresh</button>
    </div>
    
    <!-- Client-side Socket.IO and visualization logic -->
    <script src="{{ url_for('static', filename='dash.js') }}"></script>
</body>
</html>
```

```javascript
// app/static/dash.js
// Initialize Socket.IO connection
const socket = io();

// Listen for chart update events
socket.on("update_chart", (jsonData) => {
    const chartData = JSON.parse(jsonData);
    Plotly.react("chart", chartData.data, chartData.layout);
});

// Handle connection events
socket.on("connect", () => {
    console.log("Connected to server");
});

socket.on("disconnect", () => {
    console.log("Disconnected from server");
});

// Set up UI interactions
document.getElementById("refresh").addEventListener("click", () => {
    socket.emit("refresh_data");
});
```

## Summary

Key takeaways from this core application structure:
1. Separate HTTP routes from WebSocket event handlers for cleaner code organization
2. Use the application factory pattern to facilitate testing and configuration
3. Implement a service layer to handle data processing independently from communication
4. Structure logging as JSON for better observability and integration with monitoring tools
5. Maintain a single source of truth on the server with reactive updates to clients
6. Organize the client-side code to handle WebSocket events and update visualizations

## Next Steps

In the next sub-topic, we'll explore data processing and visualization techniques:
- Working with pandas for data transformation
- Creating interactive visualizations with Plotly
- Implementing efficient data streaming patterns
- Adding real-time filters and controls

---

[<- Back to Main Topic](./17-flask-socketio.md) | [Next Sub-Topic: Data Processing and Visualization ->](./17b-data-processing.md)
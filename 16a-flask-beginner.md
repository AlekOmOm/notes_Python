# 16a. Flask Beginner 🔰

[<- Back to Main Topic](./16-flask.md) | [Next Sub-Topic: Flask Intermediate ->](./16b-flask-intermediate.md)

## Overview

This sub-topic covers the essentials of getting started with Flask. You'll learn how to set up a minimal Flask application, understand the request-response cycle, and implement basic routing and views. This foundation prepares you for building more complex applications in the intermediate section.

## Key Concepts

### Setting Up a Flask Environment

```python
# Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install Flask with async support
pip install "flask[async]"

# Record dependencies
pip freeze > requirements.txt
```

The virtual environment isolates project dependencies, ensuring consistent development and deployment environments.

### Minimal Application Structure

```
mysite/
├─ app.py              # Application entry point
└─ requirements.txt    # Project dependencies
```

This simple structure is perfect for learning and prototyping. As the application grows, you can adopt a more modular structure as covered in the intermediate section.

## Implementation Patterns

### Pattern 1: The Minimal Flask Application

```python
# app.py
from flask import Flask, jsonify

# Create the central Flask application object
app = Flask(__name__)

# Define a route and its handler function
@app.get("/")
def hello():
    return jsonify(msg="Hello, Flask!")

# For direct execution
if __name__ == "__main__":
    app.run(debug=True)
```

**When to use this pattern:**
- Learning Flask basics
- Creating quick prototypes
- Building simple microservices
- Testing concepts before implementation

### Pattern 2: Environment Configuration

```python
# Setting environment variables
export FLASK_APP=app:app       # Module:variable
export FLASK_ENV=development   # Sets development mode

# Running with the Flask CLI
flask run --debug               # Enables auto-reload and debugger
```

**When to use this pattern:**
- Development workflow with auto-reload
- Running Flask from command line
- Separating configuration from code

### Pattern 3: Request Handling

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/echo", methods=["POST"])
def echo():
    data = request.get_json()
    return jsonify(received=data)

@app.route("/greet/<n>")
def greet(name):
    return f"Hello, {name}!"

@app.route("/query")
def query():
    name = request.args.get("name", "Guest")
    return f"Hello, {name}!"
```

**When to use this pattern:**
- Processing HTTP methods (GET, POST, etc.)
- Handling URL parameters
- Processing query strings
- Working with JSON payloads

## Common Challenges and Solutions

### Challenge 1: Debug Mode in Production

**Solution:**
```python
# app.py
import os
from flask import Flask

app = Flask(__name__)

# Use environment variables to control debug mode
debug_mode = os.environ.get("FLASK_DEBUG", "False").lower() == "true"

if __name__ == "__main__":
    app.run(debug=debug_mode)
```

### Challenge 2: Large Request Payloads

**Solution:**
```python
# app.py
from flask import Flask, request

app = Flask(__name__)
app.config["MAX_CONTENT_LENGTH"] = 16 * 1024 * 1024  # 16MB limit

@app.route("/upload", methods=["POST"])
def upload():
    if "file" not in request.files:
        return "No file part", 400
    
    file = request.files["file"]
    if file.filename == "":
        return "No selected file", 400
        
    # Process file...
    return "File uploaded successfully"
```

## Practical Example

Here's a complete example of a Flask application that demonstrates core concepts:

```python
from flask import Flask, request, jsonify, render_template, redirect, url_for, flash
import os

app = Flask(__name__)
app.secret_key = os.environ.get("SECRET_KEY", "dev-key-change-in-production")

# In-memory data store (replace with database in real applications)
tasks = [
    {"id": 1, "title": "Learn Flask", "completed": False},
    {"id": 2, "title": "Build a web app", "completed": False}
]

# HTML route for homepage
@app.route("/")
def index():
    return render_template("index.html", tasks=tasks)

# API routes for tasks
@app.route("/api/tasks", methods=["GET"])
def get_tasks():
    return jsonify(tasks)

@app.route("/api/tasks", methods=["POST"])
def add_task():
    if not request.json or "title" not in request.json:
        return jsonify({"error": "Title is required"}), 400
    
    task = {
        "id": max(task["id"] for task in tasks) + 1 if tasks else 1,
        "title": request.json["title"],
        "completed": request.json.get("completed", False)
    }
    
    tasks.append(task)
    return jsonify(task), 201

@app.route("/api/tasks/<int:task_id>", methods=["PUT"])
def update_task(task_id):
    task = next((t for t in tasks if t["id"] == task_id), None)
    if task is None:
        return jsonify({"error": "Task not found"}), 404
    
    task["completed"] = request.json.get("completed", task["completed"])
    task["title"] = request.json.get("title", task["title"])
    
    return jsonify(task)

@app.route("/api/tasks/<int:task_id>", methods=["DELETE"])
def delete_task(task_id):
    global tasks
    tasks = [t for t in tasks if t["id"] != task_id]
    return "", 204

# Error handlers
@app.errorhandler(404)
def not_found(error):
    return jsonify({"error": "Not found"}), 404

@app.errorhandler(500)
def server_error(error):
    return jsonify({"error": "Server error"}), 500

if __name__ == "__main__":
    app.run(debug=True)
```

This example demonstrates:
- Route definition with different HTTP methods
- Request and response handling
- Error handling
- Basic application structure

## Summary

Key takeaways from this beginner section:
1. Flask follows a minimalist approach where you add only what you need
2. The core of a Flask app is the `Flask` object that serves as the central registry
3. Routes are defined using decorators that map URLs to Python functions
4. Flask provides tools for handling different HTTP methods and request types
5. Development mode with auto-reload and debugging makes iterative development fast
6. The application's request-response cycle is the fundamental concept to understand

## Next Steps

The next sub-topic, Flask Intermediate, builds on these basics to create more structured applications with:
- Application factories
- Blueprints for organizing code
- Database integration
- Testing strategies
- Configuration management

---

[<- Back to Main Topic](./16-flask.md) | [Next Sub-Topic: Flask Intermediate ->](./16b-flask-intermediate.md)
# 16b. Flask Intermediate 🚀

[<- Back to Main Topic](./16-flask.md) | [Previous Sub-Topic: Flask Beginner ->](./16a-flask-beginner.md)

## Overview

As Flask applications grow, their organization becomes critical for maintainability. This sub-topic covers how to structure larger Flask applications using blueprints, application factories, proper configuration management, database integration, and testing strategies. These patterns enable team collaboration and code reuse while maintaining Flask's flexibility.

## Key Concepts

### Structured Application Layout

```
mysite/
├─ app/
│  ├─ __init__.py      # Application factory
│  ├─ routes.py        # Core blueprint
│  ├─ models.py        # Database models
│  ├─ auth/            # Feature blueprint
│  │   ├─ __init__.py
│  │   └─ routes.py
│  ├─ templates/       # Jinja2 templates
│  └─ static/          # Static files
├─ config.py           # Configuration classes
├─ tests/              # Test directory
└─ wsgi.py             # Production entry point
```

This structure separates concerns, making the codebase easier to navigate and maintain. Each component has a clear responsibility, enabling team collaboration.

### Application Factory Pattern

```python
# app/__init__.py
from flask import Flask
from .routes import core
from .auth import auth
from .models import db

def create_app(config_name="default"):
    """Create and configure the Flask application."""
    app = Flask(__name__, instance_relative_config=True)
    
    # Load configuration
    app.config.from_object(f"config.{config_name}")
    app.config.from_prefixed_env()  # Override with env vars
    
    # Initialize extensions
    db.init_app(app)
    
    # Register blueprints
    app.register_blueprint(core)
    app.register_blueprint(auth, url_prefix="/auth")
    
    return app
```

The factory pattern defers creating the application until runtime, enabling different configurations for development, testing, and production.

## Implementation Patterns

### Pattern 1: Configuration Management

```python
# config.py
import os

class Config:
    """Base configuration."""
    SECRET_KEY = os.environ.get("SECRET_KEY", "dev-key-change-in-production")
    SQLALCHEMY_TRACK_MODIFICATIONS = False

class DevelopmentConfig(Config):
    """Development configuration."""
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = "sqlite:///dev.db"

class TestingConfig(Config):
    """Testing configuration."""
    TESTING = True
    SQLALCHEMY_DATABASE_URI = "sqlite:///:memory:"

class ProductionConfig(Config):
    """Production configuration."""
    SQLALCHEMY_DATABASE_URI = os.environ.get(
        "DATABASE_URL", "postgresql:///myapp"
    )

# Configuration dictionary
config = {
    "development": DevelopmentConfig,
    "testing": TestingConfig,
    "production": ProductionConfig,
    "default": DevelopmentConfig
}
```

**When to use this pattern:**
- Managing different environments (dev, test, prod)
- Keeping sensitive information out of version control
- Enabling configuration overrides via environment variables

### Pattern 2: Blueprints for Feature Organization

```python
# app/auth/__init__.py
from flask import Blueprint

auth = Blueprint("auth", __name__)

from . import routes  # Import routes after creating the blueprint

# app/auth/routes.py
from flask import render_template, redirect, url_for, flash, request
from . import auth

@auth.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        # Process login
        return redirect(url_for("core.index"))
    return render_template("auth/login.html")

@auth.route("/register", methods=["GET", "POST"])
def register():
    if request.method == "POST":
        # Process registration
        return redirect(url_for("auth.login"))
    return render_template("auth/register.html")
```

**When to use this pattern:**
- Organizing routes by feature or resource
- Enabling team collaboration on different parts of the application
- Reusing code across different projects

### Pattern 3: Database Integration with Flask-SQLAlchemy

```python
# app/models.py
from flask_sqlalchemy import SQLAlchemy
from werkzeug.security import generate_password_hash, check_password_hash

db = SQLAlchemy()

class User(db.Model):
    """User model for authentication."""
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(128))
    
    @property
    def password(self):
        raise AttributeError("password is not a readable attribute")
    
    @password.setter
    def password(self, password):
        self.password_hash = generate_password_hash(password)
    
    def verify_password(self, password):
        return check_password_hash(self.password_hash, password)
    
    def __repr__(self):
        return f"<User {self.username}>"
```

**When to use this pattern:**
- Persisting data between requests
- Implementing authentication and authorization
- Building data-driven applications

## Common Challenges and Solutions

### Challenge 1: Circular Imports

**Solution:**
```python
# app/__init__.py
from flask import Flask

db = SQLAlchemy()  # Define extension instances here

def create_app():
    app = Flask(__name__)
    
    # Initialize extensions
    db.init_app(app)
    
    # Import and register blueprints (AFTER app creation)
    from .routes import main_bp
    app.register_blueprint(main_bp)
    
    return app

# app/routes.py
from flask import Blueprint
from . import db  # Import from package, not from models

main_bp = Blueprint("main", __name__)

@main_bp.route("/")
def index():
    # Use db here
    return "Hello"
```

### Challenge 2: Testing with Application Factories

**Solution:**
```python
# tests/conftest.py
import pytest
from app import create_app, db

@pytest.fixture
def app():
    app = create_app("testing")
    
    with app.app_context():
        db.create_all()
        yield app
        db.drop_all()

@pytest.fixture
def client(app):
    return app.test_client()

# tests/test_auth.py
def test_login_page(client):
    response = client.get("/auth/login")
    assert response.status_code == 200
    assert b"Login" in response.data
```

## Practical Example

Here's a complete structured Flask application example:

```python
# app/__init__.py
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

# Initialize extensions
db = SQLAlchemy()
migrate = Migrate()

def create_app(config_name="default"):
    app = Flask(__name__)
    
    # Load configuration
    from config import config
    app.config.from_object(config[config_name])
    
    # Initialize extensions with app
    db.init_app(app)
    migrate.init_app(app, db)
    
    # Register blueprints
    from .main import main as main_blueprint
    app.register_blueprint(main_blueprint)
    
    from .api import api as api_blueprint
    app.register_blueprint(api_blueprint, url_prefix="/api/v1")
    
    from .auth import auth as auth_blueprint
    app.register_blueprint(auth_blueprint, url_prefix="/auth")
    
    # Register error handlers
    from .errors import register_error_handlers
    register_error_handlers(app)
    
    # Shell context processor
    @app.shell_context_processor
    def make_shell_context():
        return dict(app=app, db=db, User=User, Post=Post)
    
    return app

# Import models to make them available
from .models import User, Post

# wsgi.py
from app import create_app

app = create_app("production")

# app/models.py
from . import db
from datetime import datetime

class User(db.Model):
    __tablename__ = "users"
    
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True, index=True)
    email = db.Column(db.String(120), unique=True, index=True)
    posts = db.relationship("Post", backref="author", lazy="dynamic")
    
    def __repr__(self):
        return f"<User {self.username}>"

class Post(db.Model):
    __tablename__ = "posts"
    
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(128))
    body = db.Column(db.Text)
    timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)
    user_id = db.Column(db.Integer, db.ForeignKey("users.id"))
    
    def __repr__(self):
        return f"<Post {self.title}>"

# app/main/__init__.py
from flask import Blueprint

main = Blueprint("main", __name__)

from . import routes

# app/main/routes.py
from flask import render_template, request, current_app
from . import main
from .. import db
from ..models import Post

@main.route("/")
def index():
    page = request.args.get("page", 1, type=int)
    pagination = Post.query.order_by(Post.timestamp.desc()).paginate(
        page=page, 
        per_page=current_app.config["POSTS_PER_PAGE"],
        error_out=False
    )
    posts = pagination.items
    return render_template("index.html", posts=posts, pagination=pagination)
```

This example demonstrates:
- Using application factory pattern
- Organizing code with blueprints
- Database integration with SQLAlchemy
- Proper configuration management
- Blueprint-based feature organization

## Summary

Key takeaways from this intermediate section:
1. Application factories enable flexible configuration and testing
2. Blueprints provide a modular structure for larger applications
3. Proper configuration management separates environments
4. SQLAlchemy integration provides robust database access
5. Testing becomes easier with proper application structure

## Next Steps

After mastering these intermediate concepts, consider exploring:
- Flask extensions for specific functionality (Flask-Login, Flask-Admin, etc.)
- Advanced API development with Flask-RESTX or Flask-Smorest
- Performance optimization and caching
- Deployment considerations for production environments
- Integration with front-end frameworks

---

[<- Back to Main Topic](./16-flask.md) | [Previous Sub-Topic: Flask Beginner ->](./16a-flask-beginner.md)
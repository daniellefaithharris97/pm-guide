# Code Reusability Best Practices

## Overview
This document outlines best practices for creating reusable code components that can be shared across teams and projects. Reusable code reduces duplication, improves maintainability, and accelerates development by leveraging proven patterns.

---

## Core Principles of Reusable Code

### **1. Modularity (Single Responsibility Principle)**

Each module or function should do one thing well.

#### **Bad Example (Not Reusable)**
```python
def process_user_order(user_id, items, payment_info):
    # Validates user
    if not check_user_exists(user_id):
        return "Invalid user"
    
    # Calculates total
    total = 0
    for item in items:
        total += item['price'] * (1 - item['discount'])
    
    # Processes payment
    charge_card(payment_info, total)
    
    # Sends email
    send_confirmation_email(user_id, items, total)
    
    # Updates inventory
    for item in items:
        update_stock(item['id'], -1)
    
    return "Success"
```

**Problems:**
- Does too many things
- Can't reuse discount logic elsewhere
- Can't reuse email logic elsewhere
- Hard to test individual pieces

#### **Good Example (Modular & Reusable)**
```python
# Reusable discount calculation
def calculate_discount(price, discount_rate):
    """Calculate discounted price"""
    return price * (1 - discount_rate)

# Reusable order total calculation
def calculate_order_total(items):
    """Calculate total from items with discounts"""
    total = 0
    for item in items:
        discounted_price = calculate_discount(item['price'], item['discount'])
        total += discounted_price * item['quantity']
    return total

# Reusable email sender
def send_order_confirmation(user_email, order_id, total):
    """Send order confirmation email"""
    subject = f"Order Confirmation #{order_id}"
    body = f"Your order total is ${total:.2f}"
    send_email(user_email, subject, body)

# Reusable inventory updater
def update_inventory(item_id, quantity_change):
    """Update item stock quantity"""
    current_stock = get_current_stock(item_id)
    new_stock = current_stock + quantity_change
    save_stock(item_id, new_stock)
    logger.info(f"Updated inventory for {item_id}: {quantity_change}")

# Main orchestration (uses reusable components)
def process_user_order(user_id, items, payment_info):
    """Process a user order using reusable components"""
    # Each step uses a focused, reusable function
    validate_user(user_id)
    total = calculate_order_total(items)
    process_payment(payment_info, total)
    order_id = create_order_record(user_id, items, total)
    send_order_confirmation(get_user_email(user_id), order_id, total)
    
    for item in items:
        update_inventory(item['id'], -item['quantity'])
    
    return order_id
```

**Benefits:**
- ✅ Each function has one clear purpose
- ✅ Can reuse `calculate_discount()` in returns, promotions, bulk orders
- ✅ Can reuse `send_order_confirmation()` for different order types
- ✅ Easy to test each function independently
- ✅ Easy to maintain and modify

---

### **2. Abstraction (Hide Implementation Details)**

Good reusable code hides complexity behind a clear, stable interface.

#### **Example: Database Access Abstraction**

**Bad (No Abstraction):**
```python
# Scattered throughout codebase
import psycopg2

def get_user_profile(user_id):
    conn = psycopg2.connect("dbname=mydb user=admin password=secret")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
    result = cursor.fetchone()
    conn.close()
    return result

def get_user_orders(user_id):
    conn = psycopg2.connect("dbname=mydb user=admin password=secret")  # Duplicated!
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM orders WHERE user_id = %s", (user_id,))
    results = cursor.fetchall()
    conn.close()
    return results
```

**Problems:**
- Connection string duplicated everywhere
- Can't switch databases easily
- No error handling
- No connection pooling
- Hard to mock for testing

**Good (With Abstraction):**
```python
# database.py - Reusable database abstraction
from contextlib import contextmanager
import psycopg2
from psycopg2 import pool
import logger

class Database:
    def __init__(self, config):
        self.connection_pool = psycopg2.pool.SimpleConnectionPool(
            1, 20,
            host=config['host'],
            database=config['database'],
            user=config['user'],
            password=config['password']
        )
        logger.info("Database connection pool initialized")
    
    @contextmanager
    def get_connection(self):
        """Get database connection from pool"""
        conn = self.connection_pool.getconn()
        try:
            yield conn
            conn.commit()
        except Exception as e:
            conn.rollback()
            logger.error(f"Database error: {e}")
            raise
        finally:
            self.connection_pool.putconn(conn)
    
    def query_one(self, sql, params=None):
        """Execute query and return single result"""
        with self.get_connection() as conn:
            cursor = conn.cursor()
            cursor.execute(sql, params or ())
            result = cursor.fetchone()
            logger.debug(f"Query executed: {sql[:50]}...")
            return result
    
    def query_many(self, sql, params=None):
        """Execute query and return multiple results"""
        with self.get_connection() as conn:
            cursor = conn.cursor()
            cursor.execute(sql, params or ())
            results = cursor.fetchall()
            logger.debug(f"Query executed: {sql[:50]}..., rows: {len(results)}")
            return results
    
    def execute(self, sql, params=None):
        """Execute non-query statement"""
        with self.get_connection() as conn:
            cursor = conn.cursor()
            cursor.execute(sql, params or ())
            logger.debug(f"Statement executed: {sql[:50]}...")

# Usage - clean and simple
db = Database(config)

def get_user_profile(user_id):
    """Get user profile - reuses database abstraction"""
    return db.query_one("SELECT * FROM users WHERE id = %s", (user_id,))

def get_user_orders(user_id):
    """Get user orders - reuses database abstraction"""
    return db.query_many("SELECT * FROM orders WHERE user_id = %s", (user_id,))
```

**Benefits:**
- ✅ Implementation hidden behind clean interface
- ✅ Can swap database (PostgreSQL → MySQL) by changing one class
- ✅ Error handling in one place
- ✅ Connection pooling handled automatically
- ✅ Easy to mock for testing
- ✅ Logging centralized

---

### **3. Parameterization (Avoid Hard-Coding)**

Hard-coded values kill reusability. Use parameters and configuration.

#### **Bad (Hard-Coded):**
```python
def send_welcome_email(user_email):
    sender = "noreply@mycompany.com"  # Hard-coded!
    subject = "Welcome to MyCompany"  # Hard-coded!
    smtp_server = "smtp.gmail.com"    # Hard-coded!
    smtp_port = 587                   # Hard-coded!
    
    send_email(sender, user_email, subject, body, smtp_server, smtp_port)
```

**Problems:**
- Can't reuse for different companies
- Can't test with different SMTP servers
- Can't customize subject line
- Can't use in different environments (dev, staging, prod)

#### **Good (Parameterized):**
```python
# email_service.py - Reusable email service
from dataclasses import dataclass
from typing import Optional
import smtplib
from email.mime.text import MIMEText
import logger

@dataclass
class EmailConfig:
    """Email configuration"""
    smtp_host: str
    smtp_port: int
    sender: str
    username: Optional[str] = None
    password: Optional[str] = None
    use_tls: bool = True

class EmailService:
    """Reusable email service"""
    
    def __init__(self, config: EmailConfig):
        self.config = config
        logger.info(f"Email service initialized: {config.smtp_host}")
    
    def send_email(
        self, 
        to: str, 
        subject: str, 
        body: str, 
        sender: Optional[str] = None,
        **kwargs
    ):
        """
        Send email with configurable parameters
        
        Args:
            to: Recipient email
            subject: Email subject
            body: Email body
            sender: Override default sender
            **kwargs: Additional email headers
        """
        sender = sender or self.config.sender
        
        msg = MIMEText(body)
        msg['Subject'] = subject
        msg['From'] = sender
        msg['To'] = to
        
        # Add custom headers
        for key, value in kwargs.items():
            msg[key] = value
        
        try:
            with smtplib.SMTP(self.config.smtp_host, self.config.smtp_port) as server:
                if self.config.use_tls:
                    server.starttls()
                
                if self.config.username and self.config.password:
                    server.login(self.config.username, self.config.password)
                
                server.send_message(msg)
                logger.info(f"Email sent to {to}: {subject}")
        except Exception as e:
            logger.error(f"Failed to send email to {to}: {e}")
            raise
    
    def send_template_email(self, to: str, template_name: str, context: dict):
        """Send email using template"""
        template = self.load_template(template_name)
        subject = template['subject'].format(**context)
        body = template['body'].format(**context)
        self.send_email(to, subject, body)

# Configuration for different environments
DEV_CONFIG = EmailConfig(
    smtp_host="localhost",
    smtp_port=1025,
    sender="dev@localhost"
)

PROD_CONFIG = EmailConfig(
    smtp_host="smtp.sendgrid.net",
    smtp_port=587,
    sender="noreply@mycompany.com",
    username=os.getenv("SMTP_USERNAME"),
    password=os.getenv("SMTP_PASSWORD")
)

# Usage - flexible and reusable
email_service = EmailService(PROD_CONFIG)

def send_welcome_email(user_email, user_name):
    """Send welcome email - uses configurable service"""
    email_service.send_template_email(
        to=user_email,
        template_name="welcome",
        context={"user_name": user_name}
    )

def send_order_confirmation(user_email, order_id, total):
    """Send order confirmation - reuses same service"""
    email_service.send_template_email(
        to=user_email,
        template_name="order_confirmation",
        context={"order_id": order_id, "total": total}
    )
```

**Benefits:**
- ✅ Works across environments (dev, staging, prod)
- ✅ Can be reused by different teams with different configs
- ✅ Easy to test with mock SMTP server
- ✅ Configuration separate from code
- ✅ Can customize sender, templates, headers

---

### **4. Low Coupling, High Cohesion**

**Low Coupling** → Modules aren't tightly dependent on each other  
**High Cohesion** → Each module focuses on a single purpose

#### **Bad (High Coupling):**
```python
class OrderProcessor:
    def __init__(self):
        self.db = PostgreSQLDatabase()  # Tightly coupled!
        self.email = GmailSender()       # Tightly coupled!
        self.payment = StripePayment()   # Tightly coupled!
    
    def process_order(self, order):
        # Can't test without real database, Gmail, and Stripe
        # Can't swap implementations
        # Can't reuse with different services
        pass
```

#### **Good (Low Coupling via Dependency Injection):**
```python
# interfaces.py - Define contracts
from abc import ABC, abstractmethod

class DatabaseInterface(ABC):
    @abstractmethod
    def save_order(self, order):
        pass
    
    @abstractmethod
    def get_order(self, order_id):
        pass

class EmailInterface(ABC):
    @abstractmethod
    def send_email(self, to, subject, body):
        pass

class PaymentInterface(ABC):
    @abstractmethod
    def charge(self, amount, payment_info):
        pass

# Implementations
class PostgreSQLDatabase(DatabaseInterface):
    def save_order(self, order):
        # PostgreSQL-specific implementation
        pass
    
    def get_order(self, order_id):
        # PostgreSQL-specific implementation
        pass

class GmailSender(EmailInterface):
    def send_email(self, to, subject, body):
        # Gmail-specific implementation
        pass

class StripePayment(PaymentInterface):
    def charge(self, amount, payment_info):
        # Stripe-specific implementation
        pass

# Reusable, loosely coupled order processor
class OrderProcessor:
    def __init__(
        self, 
        database: DatabaseInterface,
        email_service: EmailInterface,
        payment_service: PaymentInterface
    ):
        # Dependencies injected, not hard-coded
        self.db = database
        self.email = email_service
        self.payment = payment_service
    
    def process_order(self, order):
        """Process order using injected dependencies"""
        # Charge payment
        self.payment.charge(order.total, order.payment_info)
        
        # Save to database
        self.db.save_order(order)
        
        # Send confirmation
        self.email.send_email(
            order.user_email,
            "Order Confirmed",
            f"Your order #{order.id} is confirmed"
        )
        
        return order.id

# Usage - flexible composition
processor = OrderProcessor(
    database=PostgreSQLDatabase(),
    email_service=GmailSender(),
    payment_service=StripePayment()
)

# Testing - easy to mock
test_processor = OrderProcessor(
    database=MockDatabase(),
    email_service=MockEmail(),
    payment_service=MockPayment()
)

# Different environment - just swap implementations
prod_processor = OrderProcessor(
    database=PostgreSQLDatabase(),
    email_service=SendGridSender(),  # Different email service
    payment_service=StripePayment()
)
```

**Benefits:**
- ✅ Can swap implementations without changing OrderProcessor
- ✅ Easy to test with mocks
- ✅ Can reuse OrderProcessor with different services
- ✅ Each class has single responsibility
- ✅ Changes to one service don't affect others

---

### **5. Clear Interfaces / Endpoints**

Well-designed interfaces make code discoverable and reusable.

#### **API Endpoints as Reusable Interfaces**

**Bad (Inconsistent Endpoints):**
```python
# Inconsistent, hard to reuse
@app.route('/get_user/<id>')  # Inconsistent naming
def get_user(id):
    pass

@app.route('/userOrders')  # Different naming convention
def user_orders():
    pass

@app.route('/delete-order/<order_id>')  # Yet another convention
def delete_order(order_id):
    pass
```

**Good (Consistent RESTful API):**
```python
# api/v1/users.py - Reusable user endpoints
from flask import Blueprint, jsonify, request
from services.user_service import UserService
from middleware.auth import require_auth
from middleware.validation import validate_request
from schemas.user_schema import UserSchema
import logger

users_api = Blueprint('users_api', __name__, url_prefix='/api/v1/users')
user_service = UserService()

@users_api.route('/<user_id>', methods=['GET'])
@require_auth
def get_user(user_id):
    """
    Get user by ID
    
    Returns:
        200: User object
        404: User not found
        401: Unauthorized
    """
    try:
        user = user_service.get_user(user_id)
        logger.info(f"Retrieved user: {user_id}")
        return jsonify(user), 200
    except UserNotFoundError:
        return jsonify({"error": "User not found"}), 404

@users_api.route('/<user_id>/orders', methods=['GET'])
@require_auth
def get_user_orders(user_id):
    """
    Get orders for a user
    
    Returns:
        200: List of orders
        404: User not found
        401: Unauthorized
    """
    try:
        orders = user_service.get_user_orders(user_id)
        logger.info(f"Retrieved {len(orders)} orders for user: {user_id}")
        return jsonify(orders), 200
    except UserNotFoundError:
        return jsonify({"error": "User not found"}), 404

@users_api.route('', methods=['POST'])
@require_auth
@validate_request(UserSchema)
def create_user():
    """
    Create new user
    
    Body:
        name: string
        email: string
        
    Returns:
        201: User created
        400: Invalid input
        401: Unauthorized
    """
    data = request.get_json()
    try:
        user = user_service.create_user(data)
        logger.info(f"Created user: {user['id']}")
        return jsonify(user), 201
    except ValidationError as e:
        return jsonify({"error": str(e)}), 400

# api/v1/orders.py - Reusable order endpoints
orders_api = Blueprint('orders_api', __name__, url_prefix='/api/v1/orders')

@orders_api.route('/<order_id>', methods=['GET'])
@require_auth
def get_order(order_id):
    """Get order by ID"""
    pass

@orders_api.route('/<order_id>', methods=['DELETE'])
@require_auth
def delete_order(order_id):
    """Delete order"""
    pass

# Register blueprints
app.register_blueprint(users_api)
app.register_blueprint(orders_api)
```

**Benefits:**
- ✅ Consistent naming convention (RESTful)
- ✅ Clear versioning (`/api/v1/`)
- ✅ Can be reused by web, mobile, admin panel
- ✅ Documented behavior (docstrings)
- ✅ Stable contract - clients don't break when implementation changes
- ✅ Easy to discover and understand

#### **Service Classes as Reusable Interfaces**

```python
# services/user_service.py - Business logic layer
class UserService:
    """
    Reusable user business logic
    Can be used by API, CLI, background jobs
    """
    
    def __init__(self, database=None, cache=None):
        self.db = database or Database()
        self.cache = cache or Cache()
    
    def get_user(self, user_id: str) -> dict:
        """Get user by ID with caching"""
        # Check cache first
        cached = self.cache.get(f"user:{user_id}")
        if cached:
            return cached
        
        # Get from database
        user = self.db.query_one(
            "SELECT * FROM users WHERE id = %s",
            (user_id,)
        )
        
        if not user:
            raise UserNotFoundError(f"User {user_id} not found")
        
        # Cache for next time
        self.cache.set(f"user:{user_id}", user, ttl=300)
        
        return user
    
    def create_user(self, data: dict) -> dict:
        """Create new user"""
        # Validation
        self.validate_user_data(data)
        
        # Create user
        user_id = self.db.execute(
            "INSERT INTO users (name, email) VALUES (%s, %s) RETURNING id",
            (data['name'], data['email'])
        )
        
        # Invalidate cache
        self.cache.delete("users:*")
        
        return {"id": user_id, **data}
    
    def get_user_orders(self, user_id: str) -> list:
        """Get all orders for a user"""
        return self.db.query_many(
            "SELECT * FROM orders WHERE user_id = %s",
            (user_id,)
        )

# Usage in different contexts
# 1. API endpoint
user_service = UserService()
user = user_service.get_user("123")

# 2. CLI command
user_service = UserService()
users = user_service.get_all_users()
for user in users:
    print(user['email'])

# 3. Background job
user_service = UserService()
user = user_service.get_user(user_id)
send_newsletter(user['email'])
```

---

### **6. Documentation and Naming**

Good names and docs make code discoverable and reusable.

#### **Bad Naming:**
```python
def f(x, y):  # What does this do?
    return x * y * 0.1

def process(data):  # Too vague
    pass

def x123(input):  # Meaningless
    pass
```

#### **Good Naming:**
```python
def calculate_tax(subtotal: float, tax_rate: float) -> float:
    """
    Calculate tax amount based on subtotal and rate.
    
    Args:
        subtotal: Order subtotal before tax
        tax_rate: Tax rate as decimal (e.g., 0.1 for 10%)
        
    Returns:
        Tax amount rounded to 2 decimal places
        
    Example:
        >>> calculate_tax(100.00, 0.1)
        10.00
    """
    return round(subtotal * tax_rate, 2)

def mask_phi(text: str) -> str:
    """
    Mask Protected Health Information in text.
    
    Masks SSNs, phone numbers, and dates of birth to comply
    with HIPAA requirements.
    
    Args:
        text: Input text potentially containing PHI
        
    Returns:
        Text with PHI masked with placeholder values
        
    Example:
        >>> mask_phi("SSN: 123-45-6789")
        "SSN: XXX-XX-XXXX"
        
    Note:
        This function should be called before logging any
        patient-related information.
    """
    import re
    text = re.sub(r'\b\d{3}-\d{2}-\d{4}\b', 'XXX-XX-XXXX', text)
    text = re.sub(r'\b\d{3}-\d{3}-\d{4}\b', 'XXX-XXX-XXXX', text)
    text = re.sub(r'\b\d{1,2}/\d{1,2}/\d{4}\b', '[DATE]', text)
    return text

class OrderProcessor:
    """
    Process customer orders with payment and fulfillment.
    
    This class handles the complete order lifecycle:
    1. Validate order data
    2. Process payment
    3. Create order record
    4. Send confirmation email
    5. Update inventory
    
    Usage:
        processor = OrderProcessor(database, email_service, payment_service)
        order_id = processor.process_order(order_data)
        
    Attributes:
        db: Database interface for data persistence
        email: Email service for notifications
        payment: Payment service for transaction processing
    """
    pass
```

**Documentation Checklist:**
- [ ] Clear, descriptive function/class names
- [ ] Docstring explaining purpose
- [ ] Args/params documented with types
- [ ] Return value documented
- [ ] Examples provided
- [ ] Edge cases and limitations noted
- [ ] Related functions cross-referenced

---

### **7. Separation of Concerns (Layered Architecture)**

Business logic, data access, and presentation should live in different layers.

#### **Bad (Mixed Concerns):**
```python
@app.route('/users/<user_id>/orders')
def get_user_orders(user_id):
    # Data access mixed with API logic!
    conn = psycopg2.connect("dbname=mydb...")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM orders WHERE user_id = %s", (user_id,))
    orders = cursor.fetchall()
    conn.close()
    
    # Business logic mixed in!
    total_value = sum(order['total'] for order in orders)
    
    # HTML generation mixed in!
    html = "<html><body>"
    for order in orders:
        html += f"<div>Order #{order['id']}: ${order['total']}</div>"
    html += f"<p>Total: ${total_value}</p></body></html>"
    
    return html
```

#### **Good (Separated Concerns):**

```python
# Layer 1: Data Access Layer (models/order.py)
class OrderModel:
    """Data access for orders - reusable across app"""
    
    def __init__(self, database):
        self.db = database
    
    def get_by_user(self, user_id: str) -> list:
        """Get all orders for a user"""
        return self.db.query_many(
            "SELECT * FROM orders WHERE user_id = %s",
            (user_id,)
        )
    
    def get_by_id(self, order_id: str) -> dict:
        """Get order by ID"""
        return self.db.query_one(
            "SELECT * FROM orders WHERE id = %s",
            (order_id,)
        )
    
    def create(self, order_data: dict) -> str:
        """Create new order"""
        return self.db.execute(
            "INSERT INTO orders (user_id, total, items) VALUES (%s, %s, %s) RETURNING id",
            (order_data['user_id'], order_data['total'], order_data['items'])
        )

# Layer 2: Business Logic Layer (services/order_service.py)
class OrderService:
    """Business logic for orders - reusable across interfaces"""
    
    def __init__(self, order_model, email_service):
        self.order_model = order_model
        self.email_service = email_service
    
    def get_user_orders(self, user_id: str) -> list:
        """Get orders with business logic applied"""
        orders = self.order_model.get_by_user(user_id)
        
        # Apply business logic
        for order in orders:
            order['status_display'] = self.format_status(order['status'])
            order['is_recent'] = self.is_recent_order(order['created_at'])
        
        return orders
    
    def calculate_order_totals(self, user_id: str) -> dict:
        """Calculate various totals for user's orders"""
        orders = self.order_model.get_by_user(user_id)
        
        return {
            'total_orders': len(orders),
            'total_value': sum(o['total'] for o in orders),
            'average_order': sum(o['total'] for o in orders) / len(orders) if orders else 0
        }
    
    def create_order(self, order_data: dict) -> str:
        """Create order with business logic"""
        # Validate
        self.validate_order(order_data)
        
        # Create
        order_id = self.order_model.create(order_data)
        
        # Send confirmation
        self.email_service.send_order_confirmation(
            order_data['user_email'],
            order_id,
            order_data['total']
        )
        
        return order_id
    
    @staticmethod
    def format_status(status: str) -> str:
        """Format status for display"""
        status_map = {
            'pending': 'Pending Payment',
            'paid': 'Processing',
            'shipped': 'On the Way',
            'delivered': 'Delivered'
        }
        return status_map.get(status, status)
    
    @staticmethod
    def is_recent_order(created_at) -> bool:
        """Check if order is recent (within 7 days)"""
        from datetime import datetime, timedelta
        return datetime.now() - created_at < timedelta(days=7)

# Layer 3: Presentation Layer - API (api/v1/orders.py)
from flask import Blueprint, jsonify, request
from services.order_service import OrderService

orders_api = Blueprint('orders_api', __name__, url_prefix='/api/v1')
order_service = OrderService(order_model, email_service)

@orders_api.route('/users/<user_id>/orders', methods=['GET'])
def get_user_orders_api(user_id):
    """API endpoint for user orders"""
    orders = order_service.get_user_orders(user_id)
    totals = order_service.calculate_order_totals(user_id)
    
    return jsonify({
        'orders': orders,
        'summary': totals
    })

# Layer 3: Presentation Layer - CLI (cli/orders.py)
import click
from services.order_service import OrderService

@click.command()
@click.argument('user_id')
def show_user_orders(user_id):
    """CLI command for user orders"""
    order_service = OrderService(order_model, email_service)
    orders = order_service.get_user_orders(user_id)
    totals = order_service.calculate_order_totals(user_id)
    
    # Display in terminal
    for order in orders:
        print(f"Order #{order['id']}: ${order['total']} - {order['status_display']}")
    
    print(f"\nTotal Orders: {totals['total_orders']}")
    print(f"Total Value: ${totals['total_value']:.2f}")

# Layer 3: Presentation Layer - Background Job (jobs/order_report.py)
from services.order_service import OrderService

def generate_daily_order_report():
    """Background job for daily report"""
    order_service = OrderService(order_model, email_service)
    
    # Get all users
    users = user_service.get_all_users()
    
    # Generate report for each
    for user in users:
        totals = order_service.calculate_order_totals(user['id'])
        
        if totals['total_orders'] > 0:
            send_report_email(user['email'], totals)
```

**Benefits of Separation:**
- ✅ `OrderModel` can be reused by API, CLI, background jobs
- ✅ `OrderService` can be reused by different presentation layers
- ✅ Can swap presentation layer (API → GraphQL) without changing business logic
- ✅ Can swap data layer (PostgreSQL → MongoDB) without changing business logic
- ✅ Easy to test each layer independently
- ✅ Each layer has single responsibility

---

## Reusable Component Patterns

### **Pattern 1: Reusable Validation**

```python
# validators.py - Reusable validation utilities
from typing import Any, Callable, Dict, List
import re

class ValidationError(Exception):
    """Custom validation error"""
    pass

class Validator:
    """Reusable validation framework"""
    
    @staticmethod
    def required(value: Any, field_name: str):
        """Validate field is not empty"""
        if value is None or value == "":
            raise ValidationError(f"{field_name} is required")
    
    @staticmethod
    def email(value: str, field_name: str = "Email"):
        """Validate email format"""
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        if not re.match(pattern, value):
            raise ValidationError(f"{field_name} must be a valid email")
    
    @staticmethod
    def min_length(value: str, length: int, field_name: str):
        """Validate minimum length"""
        if len(value) < length:
            raise ValidationError(f"{field_name} must be at least {length} characters")
    
    @staticmethod
    def max_length(value: str, length: int, field_name: str):
        """Validate maximum length"""
        if len(value) > length:
            raise ValidationError(f"{field_name} must be at most {length} characters")
    
    @staticmethod
    def in_range(value: float, min_val: float, max_val: float, field_name: str):
        """Validate value in range"""
        if not (min_val <= value <= max_val):
            raise ValidationError(f"{field_name} must be between {min_val} and {max_val}")
    
    @staticmethod
    def one_of(value: Any, options: List, field_name: str):
        """Validate value is one of allowed options"""
        if value not in options:
            raise ValidationError(f"{field_name} must be one of: {', '.join(map(str, options))}")

def validate_schema(data: dict, schema: Dict[str, List[Callable]]) -> None:
    """
    Validate data against schema
    
    Args:
        data: Data to validate
        schema: Dict mapping field names to list of validators
        
    Example:
        schema = {
            'name': [lambda v: Validator.required(v, 'Name'),
                    lambda v: Validator.min_length(v, 3, 'Name')],
            'email': [lambda v: Validator.required(v, 'Email'),
                     lambda v: Validator.email(v, 'Email')]
        }
        validate_schema(user_data, schema)
    """
    errors = []
    
    for field, validators in schema.items():
        value = data.get(field)
        
        for validator in validators:
            try:
                validator(value)
            except ValidationError as e:
                errors.append(str(e))
    
    if errors:
        raise ValidationError("; ".join(errors))

# Usage - reusable across app
user_schema = {
    'name': [
        lambda v: Validator.required(v, 'Name'),
        lambda v: Validator.min_length(v, 3, 'Name'),
        lambda v: Validator.max_length(v, 100, 'Name')
    ],
    'email': [
        lambda v: Validator.required(v, 'Email'),
        lambda v: Validator.email(v)
    ],
    'age': [
        lambda v: Validator.in_range(v, 18, 120, 'Age')
    ]
}

def create_user(data):
    validate_schema(data, user_schema)
    # Create user...
```

### **Pattern 2: Reusable Error Handling**

```python
# error_handler.py - Reusable error handling
from functools import wraps
import logger
from typing import Callable, Any

class AppError(Exception):
    """Base application error"""
    def __init__(self, message: str, status_code: int = 500):
        self.message = message
        self.status_code = status_code
        super().__init__(message)

class NotFoundError(AppError):
    def __init__(self, resource: str):
        super().__init__(f"{resource} not found", 404)

class ValidationError(AppError):
    def __init__(self, message: str):
        super().__init__(message, 400)

class UnauthorizedError(AppError):
    def __init__(self, message: str = "Unauthorized"):
        super().__init__(message, 401)

def handle_errors(func: Callable) -> Callable:
    """
    Decorator for consistent error handling
    Reusable across all functions
    """
    @wraps(func)
    def wrapper(*args, **kwargs) -> Any:
        try:
            return func(*args, **kwargs)
        except AppError as e:
            # Known application error
            logger.error(f"Application error in {func.__name__}: {e.message}")
            raise
        except Exception as e:
            # Unknown error
            logger.error(f"Unexpected error in {func.__name__}: {e}", exc_info=True)
            raise AppError("Internal server error", 500)
    
    return wrapper

def retry_on_failure(max_retries: int = 3, delay: float = 1.0):
    """
    Decorator for automatic retries
    Reusable for any function that might fail transiently
    """
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs) -> Any:
            import time
            
            last_exception = None
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    if attempt < max_retries - 1:
                        logger.warning(
                            f"Attempt {attempt + 1}/{max_retries} failed for {func.__name__}: {e}"
                        )
                        time.sleep(delay * (attempt + 1))  # Exponential backoff
                    else:
                        logger.error(f"All {max_retries} attempts failed for {func.__name__}")
            
            raise last_exception
        
        return wrapper
    return decorator

# Usage - reusable error handling
@handle_errors
@retry_on_failure(max_retries=3)
def fetch_user_data(user_id: str):
    """Fetch user data with automatic retry and error handling"""
    response = requests.get(f"https://api.example.com/users/{user_id}")
    if response.status_code == 404:
        raise NotFoundError("User")
    return response.json()
```

### **Pattern 3: Reusable Caching**

```python
# cache.py - Reusable caching utilities
from functools import wraps
import json
import hashlib
from typing import Callable, Any, Optional
import redis
import logger

class Cache:
    """Reusable cache interface"""
    
    def __init__(self, redis_client=None):
        self.redis = redis_client or redis.Redis(
            host='localhost',
            port=6379,
            decode_responses=True
        )
    
    def get(self, key: str) -> Optional[Any]:
        """Get value from cache"""
        try:
            value = self.redis.get(key)
            if value:
                logger.debug(f"Cache hit: {key}")
                return json.loads(value)
            logger.debug(f"Cache miss: {key}")
            return None
        except Exception as e:
            logger.error(f"Cache get error: {e}")
            return None
    
    def set(self, key: str, value: Any, ttl: int = 300):
        """Set value in cache with TTL"""
        try:
            self.redis.setex(
                key,
                ttl,
                json.dumps(value)
            )
            logger.debug(f"Cache set: {key} (TTL: {ttl}s)")
        except Exception as e:
            logger.error(f"Cache set error: {e}")
    
    def delete(self, pattern: str):
        """Delete keys matching pattern"""
        try:
            keys = self.redis.keys(pattern)
            if keys:
                self.redis.delete(*keys)
                logger.debug(f"Cache deleted: {len(keys)} keys matching {pattern}")
        except Exception as e:
            logger.error(f"Cache delete error: {e}")
    
    def invalidate(self, *keys: str):
        """Invalidate specific cache keys"""
        try:
            if keys:
                self.redis.delete(*keys)
                logger.debug(f"Cache invalidated: {len(keys)} keys")
        except Exception as e:
            logger.error(f"Cache invalidate error: {e}")

def cached(ttl: int = 300, key_prefix: str = ""):
    """
    Decorator for automatic caching
    Reusable for any function
    
    Args:
        ttl: Time to live in seconds
        key_prefix: Prefix for cache keys
    """
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs) -> Any:
            # Generate cache key from function name and args
            key_parts = [key_prefix, func.__name__]
            key_parts.extend(str(arg) for arg in args)
            key_parts.extend(f"{k}={v}" for k, v in sorted(kwargs.items()))
            
            cache_key = ":".join(key_parts)
            
            # Try to get from cache
            cache = Cache()
            cached_value = cache.get(cache_key)
            
            if cached_value is not None:
                return cached_value
            
            # Execute function
            result = func(*args, **kwargs)
            
            # Store in cache
            cache.set(cache_key, result, ttl)
            
            return result
        
        return wrapper
    return decorator

# Usage - reusable caching
@cached(ttl=600, key_prefix="user")
def get_user_profile(user_id: str):
    """Get user profile with automatic caching"""
    return db.query_one("SELECT * FROM users WHERE id = %s", (user_id,))

@cached(ttl=300, key_prefix="orders")
def get_user_orders(user_id: str):
    """Get user orders with automatic caching"""
    return db.query_many("SELECT * FROM orders WHERE user_id = %s", (user_id,))
```

---

## Project Structure for Reusability

```
project/
├── src/
│   ├── models/              # Data access layer (reusable)
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── order.py
│   │   └── base.py         # Base model with common methods
│   │
│   ├── services/            # Business logic layer (reusable)
│   │   ├── __init__.py
│   │   ├── user_service.py
│   │   ├── order_service.py
│   │   └── email_service.py
│   │
│   ├── api/                 # API layer (presentation)
│   │   ├── __init__.py
│   │   ├── v1/
│   │   │   ├── users.py
│   │   │   └── orders.py
│   │   └── middleware/      # Reusable middleware
│   │       ├── auth.py
│   │       ├── validation.py
│   │       └── logging.py
│   │
│   ├── utils/               # Reusable utilities
│   │   ├── __init__.py
│   │   ├── validators.py
│   │   ├── cache.py
│   │   ├── database.py
│   │   ├── logger.py
│   │   └── errors.py
│   │
│   ├── schemas/             # Reusable data schemas
│   │   ├── __init__.py
│   │   ├── user_schema.py
│   │   └── order_schema.py
│   │
│   └── config/              # Configuration
│       ├── __init__.py
│       ├── database.py
│       ├── email.py
│       └── cache.py
│
├── tests/                   # Tests mirror src structure
│   ├── models/
│   ├── services/
│   ├── api/
│   └── utils/
│
├── docs/                    # Documentation
│   ├── api.md              # API documentation
│   ├── services.md         # Service documentation
│   └── utils.md            # Utility documentation
│
├── requirements.txt
├── setup.py
└── README.md
```

---

## Best Practices Summary

### **DO:**
- ✅ Write small, focused functions (Single Responsibility)
- ✅ Use dependency injection (Low Coupling)
- ✅ Parameterize everything (Avoid hard-coding)
- ✅ Document clearly (Good naming + docstrings)
- ✅ Separate concerns (Layered architecture)
- ✅ Create stable interfaces (APIs, classes)
- ✅ Write comprehensive tests
- ✅ Use consistent naming conventions
- ✅ Handle errors gracefully
- ✅ Log appropriately (with Winston/structured logging)

### **DON'T:**
- ❌ Write monolithic functions
- ❌ Hard-code values
- ❌ Mix data access with business logic
- ❌ Skip documentation
- ❌ Create tight coupling between modules
- ❌ Use inconsistent naming
- ❌ Ignore error handling
- ❌ Skip tests
- ❌ Over-abstract prematurely
- ❌ Create circular dependencies

---

## Quick Reusability Checklist

Before committing code, ask:

- [ ] Can this function be used in a different context?
- [ ] Are there any hard-coded values that should be parameters?
- [ ] Is the function name clear and descriptive?
- [ ] Does it have a docstring with examples?
- [ ] Is it in the right layer (data/business/presentation)?
- [ ] Does it have a single, clear responsibility?
- [ ] Are dependencies injected, not hard-coded?
- [ ] Does it follow existing naming conventions?
- [ ] Is it tested?
- [ ] Is it logged appropriately?

---

*This document should be reviewed quarterly and updated based on new patterns and team learnings.*


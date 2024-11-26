# Design Patterns Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Creational Patterns](#creational-patterns)
   - Factory Method
   - Abstract Factory
   - Builder
   - Prototype
   - Singleton
   - Object Pool
3. [Structural Patterns](#structural-patterns)
   - Adapter
   - Bridge
   - Composite
   - Decorator
   - Facade
   - Flyweight
   - Proxy
4. [Behavioral Patterns](#behavioral-patterns)
   - Chain of Responsibility
   - Command
   - Interpreter
   - Iterator
   - Mediator
   - Memento
   - Observer
   - State
   - Strategy
   - Template Method
   - Visitor
5. [Pattern Combinations](#pattern-combinations)
   - [Before Refactoring](#before-refactoring)
   - [After Refactoring](#after-refactoring)
7. [Conclusion](#conclusion)

## Introduction
The **Gang of Four (GoF) design patterns**, introduced in *"Design Patterns: Elements of Reusable Object-Oriented Software"*, provide proven solutions to common design challenges in object-oriented programming. Divided into **Creational**, **Structural**, and **Behavioral** categories, these patterns promote code reuse, flexibility, and maintainability. Below are concise descriptions and real-world examples for each pattern.

## Creational Patterns
## 1. Factory Method
Creates objects without exposing instantiation logic.

```python
from abc import ABC, abstractmethod

class Document(ABC):
    @abstractmethod
    def create(self):
        pass

class PDFDocument(Document):
    def create(self):
        return "Created PDF document"

class WordDocument(Document):
    def create(self):
        return "Created Word document"

class DocumentFactory:
    def create_document(self, doc_type: str) -> Document:
        if doc_type == "pdf":
            return PDFDocument()
        elif doc_type == "word":
            return WordDocument()
        raise ValueError("Unknown document type")

# Usage
factory = DocumentFactory()
pdf = factory.create_document("pdf")
word = factory.create_document("word")
```

## 2. Abstract Factory
Creates families of related objects.

```python
from abc import ABC, abstractmethod

class UIFactory(ABC):
    @abstractmethod
    def create_button(self): pass
    
    @abstractmethod
    def create_checkbox(self): pass

class WindowsFactory(UIFactory):
    def create_button(self):
        return WindowsButton()
    
    def create_checkbox(self):
        return WindowsCheckbox()

class MacFactory(UIFactory):
    def create_button(self):
        return MacButton()
    
    def create_checkbox(self):
        return MacCheckbox()

class Button(ABC):
    @abstractmethod
    def paint(self): pass

class Checkbox(ABC):
    @abstractmethod
    def paint(self): pass

class WindowsButton(Button):
    def paint(self):
        return "Rendering Windows button"

class MacButton(Button):
    def paint(self):
        return "Rendering Mac button"

class WindowsCheckbox(Checkbox):
    def paint(self):
        return "Rendering Windows checkbox"

class MacCheckbox(Checkbox):
    def paint(self):
        return "Rendering Mac checkbox"

# Usage
def create_ui(factory: UIFactory):
    button = factory.create_button()
    checkbox = factory.create_checkbox()
    return button.paint(), checkbox.paint()

windows_ui = create_ui(WindowsFactory())
mac_ui = create_ui(MacFactory())
```

## 3. Builder
Constructs complex objects step by step.

```python
from dataclasses import dataclass
from typing import Optional, List

@dataclass
class Pizza:
    size: str
    cheese: bool = False
    pepperoni: bool = False
    mushrooms: bool = False
    toppings: List[str] = None

class PizzaBuilder:
    def __init__(self):
        self.reset()

    def reset(self):
        self._pizza = Pizza(size="medium")
        return self

    def size(self, size: str):
        self._pizza.size = size
        return self

    def add_cheese(self):
        self._pizza.cheese = True
        return self

    def add_pepperoni(self):
        self._pizza.pepperoni = True
        return self

    def add_mushrooms(self):
        self._pizza.mushrooms = True
        return self

    def add_toppings(self, toppings: List[str]):
        self._pizza.toppings = toppings
        return self

    def build(self) -> Pizza:
        pizza = self._pizza
        self.reset()
        return pizza

# Usage
builder = PizzaBuilder()
pizza = builder.size("large").add_cheese().add_pepperoni().build()
```

## 4. Prototype
Creates objects by cloning an existing instance.

```python
from copy import deepcopy
from typing import Dict, Any

class Prototype:
    def clone(self):
        return deepcopy(self)

class Document(Prototype):
    def __init__(self, content: str, metadata: Dict[str, Any]):
        self.content = content
        self.metadata = metadata

    def __str__(self):
        return f"Content: {self.content}, Metadata: {self.metadata}"

# Usage
template = Document("Sample content", {"author": "John", "version": 1})
doc1 = template.clone()
doc2 = template.clone()
doc2.metadata["version"] = 2
```

## 5. Singleton
Ensures a class has only one instance.

```python
class DatabaseConnection:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.initialize()
        return cls._instance
    
    def initialize(self):
        self.host = "localhost"
        self.port = 5432
        self.connected = False
    
    def connect(self):
        if not self.connected:
            print(f"Connecting to {self.host}:{self.port}")
            self.connected = True
        return self.connected
    
    def query(self, sql: str):
        if self.connected:
            return f"Executing: {sql}"
        raise RuntimeError("Not connected")

# Usage
db1 = DatabaseConnection()
db2 = DatabaseConnection()
assert db1 is db2  # Same instance
db1.connect()
print(db2.query("SELECT * FROM users"))
```

## 6. Object Pool
Manages a pool of reusable objects.

```python
from typing import List, Optional
import time

class DatabaseConnection:
    def __init__(self, id: int):
        self.id = id
        self.in_use = False
        
    def execute(self, query: str):
        return f"Connection {self.id} executing: {query}"

class ConnectionPool:
    def __init__(self, size: int):
        self.size = size
        self.connections: List[DatabaseConnection] = []
        self.initialize_pool()
        
    def initialize_pool(self):
        for i in range(self.size):
            self.connections.append(DatabaseConnection(i))
    
    def get_connection(self) -> Optional[DatabaseConnection]:
        for conn in self.connections:
            if not conn.in_use:
                conn.in_use = True
                return conn
        return None
    
    def release_connection(self, conn: DatabaseConnection):
        conn.in_use = False

# Usage
pool = ConnectionPool(2)
conn1 = pool.get_connection()
conn2 = pool.get_connection()
print(conn1.execute("SELECT 1"))
pool.release_connection(conn1)
conn3 = pool.get_connection()  # Reuses conn1
```

## Structural Patterns
A guide to common structural design patterns with Python implementations.

## 1. Adapter Pattern

Converts one interface to another that clients expect.

```python
# Original classes
class USPlug:
    def power_us(self):
        return "120V power supplied"

class EUSocket:
    def power_eu(self):
        return "230V power supplied"

# Adapter implementation
class SocketAdapter:
    def __init__(self, us_plug):
        self.plug = us_plug
    
    def power_eu(self):
        power = self.plug.power_us()
        return f"Converting {power} to 230V"

# Usage
us_plug = USPlug()
adapter = SocketAdapter(us_plug)
print(adapter.power_eu())  # "Converting 120V power supplied to 230V"
```

### Additional Examples

#### 1.1 Data Format Adapter
```python
# Legacy data source
class LegacyCSVReader:
    def read_csv(self):
        return [
            {"name": "John", "age": "30"},
            {"name": "Alice", "age": "25"}
        ]

# Modern interface expected by client
class DataAnalyzer:
    def analyze_data(self, data_array):
        return f"Analyzing {len(data_array)} records..."

# Adapter to convert CSV data to required format
class CSVDataAdapter:
    def __init__(self, csv_reader):
        self.reader = csv_reader
    
    def get_data_array(self):
        csv_data = self.reader.read_csv()
        return [[record["name"], int(record["age"])] for record in csv_data]

# Usage
legacy_reader = LegacyCSVReader()
adapter = CSVDataAdapter(legacy_reader)
analyzer = DataAnalyzer()
result = analyzer.analyze_data(adapter.get_data_array())
```
#### 1.2 Third-Party Library Adapter
```python
# Third-party payment gateway
class StripePayment:
    def process_stripe(self, amount, token):
        return f"Processing ${amount} with Stripe token: {token}"

class PayPalPayment:
    def send_payment(self, amount, email):
        return f"Sending ${amount} via PayPal to {email}"

# Unified payment interface
class PaymentProcessor:
    def process_payment(self, amount, credentials):
        pass

# Adapters for different payment methods
class StripeAdapter(PaymentProcessor):
    def __init__(self, stripe):
        self.stripe = stripe
    
    def process_payment(self, amount, credentials):
        return self.stripe.process_stripe(amount, credentials["token"])

class PayPalAdapter(PaymentProcessor):
    def __init__(self, paypal):
        self.paypal = paypal
    
    def process_payment(self, amount, credentials):
        return self.paypal.send_payment(amount, credentials["email"])

# Usage
payment_method = "stripe"
credentials = {"token": "sk_test_123"} if payment_method == "stripe" else {"email": "user@example.com"}

if payment_method == "stripe":
    processor = StripeAdapter(StripePayment())
else:
    processor = PayPalAdapter(PayPalPayment())

result = processor.process_payment(100, credentials)
```

## 2. Bridge Pattern

Separates abstraction from implementation to vary them independently.

```python
# Implementations
class DrawAPI:
    def draw_circle(self, x, y, radius):
        pass

class DrawCanvas(DrawAPI):
    def draw_circle(self, x, y, radius):
        return f"Drawing on canvas at ({x},{y}) with radius {radius}"

class DrawSVG(DrawAPI):
    def draw_circle(self, x, y, radius):
        return f"Rendering SVG circle at ({x},{y}) with radius {radius}"

# Abstraction
class Shape:
    def __init__(self, draw_api):
        self.api = draw_api

    def draw(self):
        pass

class Circle(Shape):
    def __init__(self, x, y, radius, draw_api):
        super().__init__(draw_api)
        self.x = x
        self.y = y
        self.radius = radius

    def draw(self):
        return self.api.draw_circle(self.x, self.y, self.radius)

# Usage
canvas_circle = Circle(10, 10, 5, DrawCanvas())
svg_circle = Circle(10, 10, 5, DrawSVG())
```

### Additional Examples

#### 2.1 Message Sender Bridge
```python
# Implementations
class MessageSender:
    def send(self, message):
        pass

class EmailSender(MessageSender):
    def send(self, message):
        return f"Sending email: {message}"

class SMSSender(MessageSender):
    def send(self, message):
        return f"Sending SMS: {message}"

class SlackSender(MessageSender):
    def send(self, message):
        return f"Sending Slack message: {message}"

# Abstractions
class NotificationService:
    def __init__(self, sender: MessageSender):
        self.sender = sender

    def notify(self, message):
        pass

class UrgentNotification(NotificationService):
    def notify(self, message):
        return self.sender.send(f"URGENT: {message}")

class RegularNotification(NotificationService):
    def notify(self, message):
        return self.sender.send(message)

# Usage
email_urgent = UrgentNotification(EmailSender())
sms_regular = RegularNotification(SMSSender())
slack_urgent = UrgentNotification(SlackSender())

print(email_urgent.notify("System failure"))  # "Sending email: URGENT: System failure"
print(sms_regular.notify("Daily update"))     # "Sending SMS: Daily update"
```

#### 2.2 Document Renderer Bridge
```python
# Implementations
class Renderer:
    def render_text(self, text):
        pass
    
    def render_image(self, url):
        pass

class HTMLRenderer(Renderer):
    def render_text(self, text):
        return f"<p>{text}</p>"
    
    def render_image(self, url):
        return f'<img src="{url}">'

class MarkdownRenderer(Renderer):
    def render_text(self, text):
        return text
    
    def render_image(self, url):
        return f"![image]({url})"

# Abstractions
class Document:
    def __init__(self, renderer: Renderer):
        self.renderer = renderer
    
    def render(self):
        pass

class Article(Document):
    def __init__(self, renderer, title, content, image_url):
        super().__init__(renderer)
        self.title = title
        self.content = content
        self.image_url = image_url
    
    def render(self):
        return (
            self.renderer.render_text(self.title) + "\n" +
            self.renderer.render_image(self.image_url) + "\n" +
            self.renderer.render_text(self.content)
        )

# Usage
html_article = Article(
    HTMLRenderer(),
    "My Article",
    "This is the content",
    "image.jpg"
)

markdown_article = Article(
    MarkdownRenderer(),
    "My Article",
    "This is the content",
    "image.jpg"
)
```

## 3. Composite Pattern

Composes objects into tree structures for part-whole hierarchies.

```python
class Component:
    def get_price(self):
        pass

class Product(Component):
    def __init__(self, name, price):
        self.name = name
        self.price = price

    def get_price(self):
        return self.price

class Box(Component):
    def __init__(self):
        self.items = []

    def add(self, item):
        self.items.append(item)

    def get_price(self):
        return sum(item.get_price() for item in self.items)

# Usage
phone = Product("Phone", 500)
charger = Product("Charger", 30)
headphones = Product("Headphones", 100)

package = Box()
package.add(phone)
electronics_box = Box()
electronics_box.add(charger)
electronics_box.add(headphones)
package.add(electronics_box)

print(package.get_price())  # 630
```
### Additional Examples

#### 3.1 File System Structure
```python
class FileSystemComponent:
    def get_size(self):
        pass
    
    def print_structure(self, indent=""):
        pass

class File(FileSystemComponent):
    def __init__(self, name, size):
        self.name = name
        self._size = size
    
    def get_size(self):
        return self._size
    
    def print_structure(self, indent=""):
        return f"{indent}{self.name} ({self._size}B)"

class Directory(FileSystemComponent):
    def __init__(self, name):
        self.name = name
        self.children = []
    
    def add(self, component):
        self.children.append(component)
        return self
    
    def get_size(self):
        return sum(child.get_size() for child in self.children)
    
    def print_structure(self, indent=""):
        result = [f"{indent}{self.name}/"]
        for child in self.children:
            result.append(child.print_structure(indent + "  "))
        return "\n".join(result)

# Usage
root = Directory("root")
documents = Directory("documents")
pictures = Directory("pictures")

documents.add(File("resume.pdf", 1000))
documents.add(File("cover_letter.doc", 500))
pictures.add(File("vacation.jpg", 2000))
pictures.add(File("family.jpg", 1500))

root.add(documents).add(pictures)
print(root.print_structure())
print(f"Total size: {root.get_size()}B")
```

#### 3.2 Organization Structure
```python
class Employee:
    def get_salary(self):
        pass
    
    def get_details(self):
        pass

class Individual(Employee):
    def __init__(self, name, position, salary):
        self.name = name
        self.position = position
        self.salary = salary
    
    def get_salary(self):
        return self.salary
    
    def get_details(self):
        return f"{self.name} ({self.position})"

class Department(Employee):
    def __init__(self, name):
        self.name = name
        self.employees = []
    
    def add(self, employee):
        self.employees.append(employee)
        return self
    
    def get_salary(self):
        return sum(emp.get_salary() for emp in self.employees)
    
    def get_details(self):
        result = [f"Department: {self.name}"]
        for emp in self.employees:
            result.append(f"  - {emp.get_details()}")
        return "\n".join(result)

# Usage
company = Department("Tech Corp")
engineering = Department("Engineering")
marketing = Department("Marketing")

engineering.add(Individual("John Doe", "Senior Dev", 100000))
engineering.add(Individual("Jane Smith", "Dev", 80000))
marketing.add(Individual("Mike Johnson", "Marketing Manager", 90000))
marketing.add(Individual("Sarah Wilson", "Marketing Specialist", 70000))

company.add(engineering).add(marketing)
print(company.get_details())
print(f"Total payroll: ${company.get_salary()}")
```
## 4. Decorator Pattern

Adds behavior to objects dynamically.

```python
class Coffee:
    def get_cost(self):
        return 3.0
    
    def get_description(self):
        return "Plain coffee"

class CoffeeDecorator(Coffee):
    def __init__(self, coffee):
        self.coffee = coffee
    
    def get_cost(self):
        return self.coffee.get_cost()
    
    def get_description(self):
        return self.coffee.get_description()

class Milk(CoffeeDecorator):
    def get_cost(self):
        return self.coffee.get_cost() + 0.5
    
    def get_description(self):
        return f"{self.coffee.get_description()} + milk"

class Sugar(CoffeeDecorator):
    def get_cost(self):
        return self.coffee.get_cost() + 0.2
    
    def get_description(self):
        return f"{self.coffee.get_description()} + sugar"

# Usage
coffee = Coffee()
coffee_with_milk = Milk(coffee)
sweet_coffee_with_milk = Sugar(coffee_with_milk)
print(sweet_coffee_with_milk.get_description())  # "Plain coffee + milk + sugar"
print(sweet_coffee_with_milk.get_cost())  # 3.7
```
### 4. Decorator Pattern Examples

#### 4.1 Text Formatting System
```python
class Text:
    def render(self):
        pass

class PlainText(Text):
    def __init__(self, text):
        self._text = text
    
    def render(self):
        return self._text

class TextDecorator(Text):
    def __init__(self, text_object):
        self._text_object = text_object
    
    def render(self):
        return self._text_object.render()

class BoldDecorator(TextDecorator):
    def render(self):
        return f"<b>{self._text_object.render()}</b>"

class ItalicDecorator(TextDecorator):
    def render(self):
        return f"<i>{self._text_object.render()}</i>"

class UnderlineDecorator(TextDecorator):
    def render(self):
        return f"<u>{self._text_object.render()}</u>"

# Usage
text = PlainText("Hello, World!")
bold_italic = ItalicDecorator(BoldDecorator(text))
underline_bold_italic = UnderlineDecorator(bold_italic)
print(underline_bold_italic.render())  # <u><i><b>Hello, World!</b></i></u>
```

#### 4.2 Data Processing Pipeline
```python
class DataSource:
    def process_data(self, data):
        pass

class BasicDataSource(DataSource):
    def process_data(self, data):
        return data

class DataDecorator(DataSource):
    def __init__(self, source):
        self._source = source
    
    def process_data(self, data):
        return self._source.process_data(data)

class UppercaseDecorator(DataDecorator):
    def process_data(self, data):
        return self._source.process_data(data).upper()

class ValidationDecorator(DataDecorator):
    def process_data(self, data):
        processed_data = self._source.process_data(data)
        if not processed_data:
            raise ValueError("Data cannot be empty")
        return processed_data

class LoggingDecorator(DataDecorator):
    def process_data(self, data):
        print(f"Processing data: {data}")
        result = self._source.process_data(data)
        print(f"Processing complete: {result}")
        return result

# Usage
source = BasicDataSource()
pipeline = LoggingDecorator(
    ValidationDecorator(
        UppercaseDecorator(source)
    )
)

try:
    result = pipeline.process_data("Hello, World!")
    print(f"Final result: {result}")
    
    # This will raise an error
    pipeline.process_data("")
except ValueError as e:
    print(f"Error: {e}")
```
## 5. Facade Pattern

Provides a simplified interface to a complex subsystem.

```python
class CPU:
    def freeze(self):
        return "CPU frozen"
    
    def execute(self):
        return "CPU executing"

class Memory:
    def load(self):
        return "Memory loaded"

class HardDrive:
    def read(self):
        return "Data read from hard drive"

class ComputerFacade:
    def __init__(self):
        self.cpu = CPU()
        self.memory = Memory()
        self.hard_drive = HardDrive()
    
    def start(self):
        operations = [
            self.cpu.freeze(),
            self.memory.load(),
            self.hard_drive.read(),
            self.cpu.execute()
        ]
        return " -> ".join(operations)

# Usage
computer = ComputerFacade()
print(computer.start())  # "CPU frozen -> Memory loaded -> Data read from hard drive -> CPU executing"
```

#### Advanced examples

### 1.1 Smart Home System Facade
```python
# Complex subsystems
class Light:
    def __init__(self, room):
        self.room = room
        self.is_on = False
    
    def turn_on(self):
        self.is_on = True
        return f"{self.room} lights are on"
    
    def turn_off(self):
        self.is_on = False
        return f"{self.room} lights are off"

class Temperature:
    def __init__(self):
        self.temperature = 21
    
    def set_temperature(self, temp):
        self.temperature = temp
        return f"Temperature set to {temp}°C"
    
    def get_temperature(self):
        return self.temperature

class Security:
    def __init__(self):
        self.armed = False
    
    def arm_system(self):
        self.armed = True
        return "Security system armed"
    
    def disarm_system(self):
        self.armed = False
        return "Security system disarmed"

class MusicPlayer:
    def __init__(self):
        self.playing = False
        self.current_track = None
    
    def play(self, track):
        self.playing = True
        self.current_track = track
        return f"Playing: {track}"
    
    def stop(self):
        self.playing = False
        return "Music stopped"

# Facade
class SmartHomeFacade:
    def __init__(self):
        self.living_room_light = Light("Living Room")
        self.bedroom_light = Light("Bedroom")
        self.temperature = Temperature()
        self.security = Security()
        self.music = MusicPlayer()
    
    def leave_home(self):
        actions = [
            self.living_room_light.turn_off(),
            self.bedroom_light.turn_off(),
            self.temperature.set_temperature(18),
            self.security.arm_system(),
            self.music.stop()
        ]
        return "\n".join(actions)
    
    def return_home(self):
        actions = [
            self.security.disarm_system(),
            self.living_room_light.turn_on(),
            self.temperature.set_temperature(21),
            self.music.play("Welcome Home Playlist")
        ]
        return "\n".join(actions)
    
    def night_mode(self):
        actions = [
            self.living_room_light.turn_off(),
            self.bedroom_light.turn_off(),
            self.temperature.set_temperature(19),
            self.security.arm_system(),
            self.music.play("Sleep Playlist")
        ]
        return "\n".join(actions)

# Usage
smart_home = SmartHomeFacade()
print("Leaving home:")
print(smart_home.leave_home())
print("\nReturning home:")
print(smart_home.return_home())
print("\nNight mode:")
print(smart_home.night_mode())
```

### 1.2 E-Commerce Order Processing Facade
```python
from datetime import datetime

class Inventory:
    def __init__(self):
        self.items = {"item1": 10, "item2": 5, "item3": 15}
    
    def check_availability(self, item_id, quantity):
        return self.items.get(item_id, 0) >= quantity
    
    def reduce_stock(self, item_id, quantity):
        if self.check_availability(item_id, quantity):
            self.items[item_id] -= quantity
            return True
        return False

class Payment:
    def process_payment(self, amount, card_details):
        # Simulate payment processing
        if card_details.get("valid_until") > datetime.now():
            return True
        return False

class Shipping:
    def calculate_shipping(self, address, items):
        # Simplified shipping calculation
        base_rate = 10
        item_rate = 2 * len(items)
        return base_rate + item_rate
    
    def create_shipping_label(self, order_id, address):
        return f"Shipping Label for Order {order_id} to {address}"

class Notification:
    def send_email(self, to_address, subject, message):
        return f"Email sent to {to_address}: {subject}"
    
    def send_sms(self, phone_number, message):
        return f"SMS sent to {phone_number}: {message}"

# Facade
class OrderProcessingFacade:
    def __init__(self):
        self.inventory = Inventory()
        self.payment = Payment()
        self.shipping = Shipping()
        self.notification = Notification()
    
    def process_order(self, order):
        """
        Order format:
        {
            'order_id': '12345',
            'items': [{'item_id': 'item1', 'quantity': 2}, ...],
            'customer': {
                'email': 'customer@example.com',
                'phone': '1234567890',
                'address': '123 Main St',
                'card_details': {'number': '1234', 'valid_until': datetime(...)}
            }
        }
        """
        results = []
        
        # Check inventory
        for item in order['items']:
            if not self.inventory.check_availability(item['item_id'], item['quantity']):
                return f"Item {item['item_id']} is out of stock"
        
        # Calculate total and shipping
        total = sum(item['quantity'] * 10 for item in order['items'])  # Simplified pricing
        shipping_cost = self.shipping.calculate_shipping(
            order['customer']['address'],
            order['items']
        )
        
        # Process payment
        if not self.payment.process_payment(total + shipping_cost, 
                                         order['customer']['card_details']):
            return "Payment processing failed"
        
        # Update inventory
        for item in order['items']:
            self.inventory.reduce_stock(item['item_id'], item['quantity'])
        
        # Create shipping label
        shipping_label = self.shipping.create_shipping_label(
            order['order_id'],
            order['customer']['address']
        )
        
        # Send notifications
        self.notification.send_email(
            order['customer']['email'],
            "Order Confirmed",
            f"Your order {order['order_id']} has been processed"
        )
        
        self.notification.send_sms(
            order['customer']['phone'],
            f"Order {order['order_id']} is being processed"
        )
        
        return {
            'status': 'success',
            'order_id': order['order_id'],
            'total': total + shipping_cost,
            'shipping_label': shipping_label
        }

# Usage
facade = OrderProcessingFacade()
order = {
    'order_id': '12345',
    'items': [
        {'item_id': 'item1', 'quantity': 2},
        {'item_id': 'item2', 'quantity': 1}
    ],
    'customer': {
        'email': 'customer@example.com',
        'phone': '1234567890',
        'address': '123 Main St',
        'card_details': {
            'number': '1234',
            'valid_until': datetime(2025, 12, 31)
        }
    }
}

result = facade.process_order(order)
print(result)
```

## 6. Flyweight Pattern

Minimizes memory usage by sharing data across similar objects.

```python
class Character:
    def __init__(self, char):
        self.char = char
        # Assume each character consumes significant memory
        self.style = {"font": "Arial", "size": 12}

class TextEditor:
    def __init__(self):
        self._characters = {}
    
    def get_character(self, char):
        if char not in self._characters:
            self._characters[char] = Character(char)
        return self._characters[char]
    
    def write_text(self, text):
        return [self.get_character(c) for c in text]
    
    def memory_usage(self):
        return len(self._characters)

# Usage
editor = TextEditor()
editor.write_text("hello world")
print(editor.memory_usage())  # 8 (unique characters: h,e,l,o,w,r,d, )
```
#### Advanced examples
Minimizes memory usage by sharing as much data as possible with similar objects.

### 2.1 Text Editor Character Flyweight
```python
class CharacterStyle:
    def __init__(self, font, size, bold, italic):
        self.font = font
        self.size = size
        self.bold = bold
        self.italic = italic
    
    def __eq__(self, other):
        return (self.font == other.font and 
                self.size == other.size and 
                self.bold == other.bold and 
                self.italic == other.italic)
    
    def __hash__(self):
        return hash((self.font, self.size, self.bold, self.italic))

class Character:
    def __init__(self, char, style):
        self.char = char
        self.style = style

class StyleFactory:
    _styles = {}
    
    @classmethod
    def get_style(cls, font, size, bold=False, italic=False):
        key = CharacterStyle(font, size, bold, italic)
        if key not in cls._styles:
            cls._styles[key] = key
        return cls._styles[key]
    
    @classmethod
    def get_style_count(cls):
        return len(cls._styles)

class TextEditor:
    def __init__(self):
        self.characters = []
    
    def add_character(self, char, font, size, bold=False, italic=False):
        style = StyleFactory.get_style(font, size, bold, italic)
        self.characters.append(Character(char, style))
    
    def get_text(self):
        return "".join(c.char for c in self.characters)
    
    def get_memory_usage(self):
        # Simplified memory calculation
        chars_memory = len(self.characters) * 2  # 2 bytes per character
        styles_memory = StyleFactory.get_style_count() * 10  # 10 bytes per style
        return chars_memory + styles_memory

# Usage
editor = TextEditor()

# Adding text with different styles
text = "Hello, World!"
for char in text:
    if char.isupper():
        editor.add_character(char, "Arial", 12, bold=True)
    else:
        editor.add_character(char, "Arial", 12)

print(f"Text: {editor.get_text()}")
print(f"Memory usage: {editor.get_memory_usage()} bytes")
print(f"Unique styles: {StyleFactory.get_style_count()}")
```

### 2.2 Game Terrain Flyweight
```python
from typing import Dict, Tuple
import random

class TerrainType:
    def __init__(self, name, texture, buildable, movement_cost):
        self.name = name
        self.texture = texture  # Simulated texture data
        self.buildable = buildable
        self.movement_cost = movement_cost
    
    def __eq__(self, other):
        return (self.name == other.name and 
                self.texture == other.texture and 
                self.buildable == other.buildable and 
                self.movement_cost == other.movement_cost)
    
    def __hash__(self):
        return hash((self.name, self.texture, self.buildable, self.movement_cost))

class TerrainFactory:
    _terrain_types: Dict[str, TerrainType] = {}
    
    @classmethod
    def get_terrain_type(cls, name, texture, buildable, movement_cost):
        key = name
        if key not in cls._terrain_types:
            cls._terrain_types[key] = TerrainType(name, texture, buildable, movement_cost)
        return cls._terrain_types[key]
    
    @classmethod
    def get_type_count(cls):
        return len(cls._terrain_types)

class TerrainTile:
    def __init__(self, terrain_type: TerrainType, x: int, y: int):
        self.terrain_type = terrain_type
        self.x = x
        self.y = y
        self.unit = None  # Current unit on this tile

class GameMap:
    def __init__(self, width: int, height: int):
        self.width = width
        self.height = height
        self.tiles: Dict[Tuple[int, int], TerrainTile] = {}
        self.terrain_factory = TerrainFactory()
    
    def initialize_random_map(self):
        terrain_configs = {
            'grass': ('grass_texture', True, 1),
            'water': ('water_texture', False, 2),
            'mountain': ('mountain_texture', False, 3),
            'forest': ('forest_texture', True, 2),
            'desert': ('desert_texture', True, 1.5)
        }
        
        for x in range(self.width):
            for y in range(self.height):
                terrain_name = random.choice(list(terrain_configs.keys()))
                texture, buildable, cost = terrain_configs[terrain_name]
                terrain_type = TerrainFactory.get_terrain_type(
                    terrain_name, texture, buildable, cost
                )
                self.tiles[(x, y)] = TerrainTile(terrain_type, x, y)
    
    def get_tile(self, x: int, y: int) -> TerrainTile:
        return self.tiles.get((x, y))
    
    def get_movement_cost(self, x: int, y: int) -> float:
        tile = self.get_tile(x, y)
        return tile.terrain_type.movement_cost if tile else float('inf')
    
    def is_buildable(self, x: int, y: int) -> bool:
        tile = self.get_tile(x, y)
        return tile.terrain_type.buildable if tile else False
    
    def get_memory_usage(self):
        # Simplified memory calculation
        tile_memory = len(self.tiles) * 12  # 12 bytes per tile (coordinates and reference)
        terrain_memory = TerrainFactory.get_type_count() * 100  # 100 bytes per terrain type
        return tile_memory + terrain_memory

# Usage
game_map = GameMap(100, 100)  # 10000 tiles
game_map.initialize_random_map()

print(f"Map size: {game_map.width}x{game_map.height}")
print(f"Memory usage: {game_map.get_memory_usage()} bytes")
print(f"Unique terrain types: {TerrainFactory.get_type_count()}")

# Simulate pathfinding cost calculation
start_x, start_y = 0, 0
end_x, end_y = 5, 5
total_cost = sum(
    game_map.get_movement_cost(x, y)
    for x in range(start_x, end_x + 1)
    for y in range(start_y, end_y + 1)
)
print(f"Path cost from ({start_x},{start_y}) to ({end_x},{end_y}): {total_cost}")
```
## 7. Proxy Pattern

Controls access to an object through a surrogate object.

```python
from time import sleep

class RemoteService:
    def execute(self):
        sleep(2)  # Simulating slow network call
        return "Remote operation completed"

class RemoteProxy:
    def __init__(self):
        self.service = None
        self.cached_result = None
    
    def execute(self):
        if not self.service:
            self.service = RemoteService()
        
        if not self.cached_result:
            self.cached_result = self.service.execute()
        
        return self.cached_result

# Usage
proxy = RemoteProxy()
print(proxy.execute())  # Slow first call
print(proxy.execute())  # Fast subsequent calls
```
#### More Advanced examples
### 3.1 Cached Database Proxy
```python
from typing import Dict, Optional
from datetime import datetime, timedelta

class Database:
    def __init__(self):
        # Simulate a real database
        self._data = {}
        self.query_count = 0
    
    def query(self, sql: str) -> Dict:
        self.query_count += 1
        # Simulate database query
        if sql.startswith("SELECT"):
            return self._data.get(sql, {"result": "No data"})
        elif sql.startswith("INSERT"):
            self._data[sql] = {"result": "Data inserted"}
            return {"status": "success"}
        return {"error": "Invalid query"}

class CachedDatabaseProxy:
    def __init__(self, database: Database):
        self._database = database
        self._cache: Dict[str, tuple[Dict, datetime]] = {}
        self._cache_duration = timedelta(minutes=30)
    
    def query(self, sql: str) -> Dict:
        # Don't cache write operations
        if not sql.startswith("SELECT"):
            return self._database.query(sql)
        
        # Check cache
        if sql in self._cache:
            result, timestamp = self._cache[sql]
            if datetime.now() - timestamp < self._cache_duration:
                return result.copy()  # Return cached copy
        
        # Query database and update cache
        result = self._database.query(sql)
        self._cache[sql] = (result.copy(), datetime.now())
        return result
    
    def get_cache_stats(self):
        return {
            "cache_size": len(self._cache),
            "database_queries": self._database.query_count
        }
    
    def clear_cache(self):
        self._cache.clear()
        return "Cache cleared"

# Usage
db = Database()
proxy = CachedDatabaseProxy(db)

# First query - will hit database
result1 = proxy.query("SELECT * FROM users WHERE id = 1")
print("First query result:", result1)

# Second query - will use cache
result2 = proxy.query("SELECT * FROM users WHERE id = 1")
print("Second query result:", result2)

# Check stats
print("Cache stats:", proxy.get_cache_stats())
```

### 3.2 Access Control Proxy
```python
from enum import Enum
from typing import Optional, Dict, List
from datetime import datetime

class AccessLevel(Enum):
    READ = 1
    WRITE = 2
    ADMIN = 3

class User:
    def __init__(self, username: str, access_level: AccessLevel):
        self.username = username
        self.access_level = access_level

class Document:
    def __init__(self, title: str, content: str):
        self.title = title
        self.content = content
        self.created_at = datetime.now()
        self.modified_at = datetime.now()

class DocumentManager:
    def __init__(self):
        self._documents: Dict[str, Document] = {}
    
    def create_document(self, title: str, content: str) -> bool:
        self._documents[title] = Document(title, content)
        return True
    
    def read_document(self, title: str) -> Optional[Document]:
        return self._documents.get(title)
    
    def update_document(self, title: str, content: str) -> bool:
        if title in self._documents:
            doc = self._documents[title]
            doc.content = content
            doc.modified_at = datetime.now()
            return True
        return False
    
    def delete_document(self, title: str) -> bool:
        if title in self._documents:
            del self._documents[title]
            return True
        return False

class DocumentManagerProxy:
    def __init__(self, document_manager: DocumentManager):
        self._manager = document_manager
        self._current_user: Optional[User] = None
        self._access_log: List[str] = []
    
    def login(self, user: User):
        self._current_user = user
        self._log_access(f"User {user.username} logged in")
    
    def logout(self):
        if self._current_user:
            self._log_access(f"User {self._current_user.username} logged out")
            self._current_user = None
    
    def _log_access(self, message: str):
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        self._access_log.append(f"[{timestamp}] {message}")
    
    def _check_access(self, required_level: AccessLevel) -> bool:
        if not self._current_user:
            raise ValueError("No user logged in")
        return self._current_user.access_level.value >= required_level.value
    
    def create_document(self, title: str, content: str) -> bool:
        if self._check_access(AccessLevel.WRITE):
            result = self._manager.create_document(title, content)
            self._log_access(f"User {self._current_user.username} created document: {title}")
            return result
        raise PermissionError("Insufficient permissions to create document")
    
    def read_document(self, title: str) -> Optional[Document]:
        if self._check_access(AccessLevel.READ):
            result = self._manager.read_document(title)
            if result:
                self._log_access(f"User {self._current_user.username} read document: {title}")
            return result
        raise PermissionError("Insufficient permissions to read document")
    
    def update_document(self, title: str, content: str) -> bool:
        if self._check_access(AccessLevel.WRITE):
            result = self._manager.update_document(title, content)
            if result:
                self._log_access(f"User {self._current_user.username} updated document: {title}")
            return result
        raise PermissionError("Insufficient permissions to update document")
    
    def delete_document(self, title: str) -> bool:
        if self._check_access(AccessLevel.ADMIN):
            result = self._manager.delete_document(title)
            if result:
                self._log_access(f"User {self._current_user.username} deleted document: {title}")
            return result
        raise PermissionError("Insufficient permissions to delete document")
    
    def get_access_log(self) -> List[str]:
        if self._check_access(AccessLevel.ADMIN):
            return self._access_log.copy()
        raise PermissionError("Insufficient permissions to view access log")

# Usage
doc_manager = DocumentManager()
proxy = DocumentManagerProxy(doc_manager)

# Create users with different access levels
reader = User("john", AccessLevel.READ)
editor = User("alice", AccessLevel.WRITE)
admin = User("admin", AccessLevel.ADMIN)

# Test document operations with different users
try:
    # Reader operations
    proxy.login(reader)
    proxy.read_document("test.txt")  # Should work
    try:
        proxy.create_document("test.txt", "content")  # Should fail
    except PermissionError as e:
        print(f"Permission error: {e}")
    proxy.logout()

    # Editor operations
    proxy.login(editor)
    proxy.create_document("test.txt", "Hello, World!")  # Should work
    proxy.update_document("test.txt", "Updated content")  # Should work
    try:
        proxy.delete_document("test.txt")  # Should fail
    except PermissionError as e:
        print(f"Permission error: {e}")
    proxy.logout()

    # Admin operations
    proxy.login(admin)
    print("\nAccess Log:")
    for entry in proxy.get_access_log():
        print(entry)
    proxy.delete_document("test.txt")  # Should work
    proxy.logout()

except Exception as e:
    print(f"Error: {e}")
```
## Behavioral Patterns
## 1. Chain of Responsibility
Passes requests along a chain of handlers.

```python
class Logger:
    def __init__(self, level, next_logger=None):
        self.level = level
        self.next = next_logger

    def log(self, level, message):
        if level >= self.level:
            self._write(message)
        if self.next:
            self.next.log(level, message)

    def _write(self, message):
        pass

class ConsoleLogger(Logger):
    def _write(self, message):
        print(f"Console: {message}")

class FileLogger(Logger):
    def _write(self, message):
        print(f"File: {message}")

class EmailLogger(Logger):
    def _write(self, message):
        print(f"Email: {message}")

# Usage
logger = ConsoleLogger(1, FileLogger(2, EmailLogger(3)))
logger.log(1, "Debug info")  # Console only
logger.log(2, "System error")  # Console and File
logger.log(3, "Critical failure")  # All three
```
### 1.2 Request Processing Pipeline
```python
from abc import ABC, abstractmethod
from typing import Optional, Dict
from datetime import datetime

class Request:
    def __init__(self, user_id: str, resource: str, action: str):
        self.user_id = user_id
        self.resource = resource
        self.action = action
        self.timestamp = datetime.now()
        self.metadata: Dict = {}

class Handler(ABC):
    def __init__(self):
        self._next_handler: Optional[Handler] = None
    
    def set_next(self, handler: 'Handler') -> 'Handler':
        self._next_handler = handler
        return handler
    
    def handle(self, request: Request) -> Optional[str]:
        result = self.process_request(request)
        if result is None and self._next_handler:
            return self._next_handler.handle(request)
        return result
    
    @abstractmethod
    def process_request(self, request: Request) -> Optional[str]:
        pass

class AuthenticationHandler(Handler):
    def __init__(self):
        super().__init__()
        self._valid_users = {"user1", "user2", "admin"}
    
    def process_request(self, request: Request) -> Optional[str]:
        if request.user_id not in self._valid_users:
            return f"Authentication failed for user {request.user_id}"
        request.metadata['authenticated'] = True
        return None

class AuthorizationHandler(Handler):
    def __init__(self):
        super().__init__()
        self._permissions = {
            "user1": {"read"},
            "user2": {"read", "write"},
            "admin": {"read", "write", "delete"}
        }
    
    def process_request(self, request: Request) -> Optional[str]:
        if not request.metadata.get('authenticated'):
            return "Authorization skipped - not authenticated"
        
        allowed_actions = self._permissions.get(request.user_id, set())
        if request.action not in allowed_actions:
            return f"User {request.user_id} not authorized for {request.action}"
        
        request.metadata['authorized'] = True
        return None

class RateLimitHandler(Handler):
    def __init__(self, requests_per_minute: int = 60):
        super().__init__()
        self._requests_per_minute = requests_per_minute
        self._request_timestamps: Dict[str, list] = {}
    
    def process_request(self, request: Request) -> Optional[str]:
        user_timestamps = self._request_timestamps.setdefault(request.user_id, [])
        current_time = datetime.now()
        
        # Remove timestamps older than 1 minute
        user_timestamps = [ts for ts in user_timestamps 
                         if (current_time - ts).total_seconds() < 60]
        
        if len(user_timestamps) >= self._requests_per_minute:
            return f"Rate limit exceeded for user {request.user_id}"
        
        user_timestamps.append(current_time)
        self._request_timestamps[request.user_id] = user_timestamps
        return None

class LoggingHandler(Handler):
    def process_request(self, request: Request) -> Optional[str]:
        print(f"[{request.timestamp}] {request.user_id} -> {request.action} {request.resource}")
        return None

# Usage
def create_request_pipeline() -> Handler:
    authentication = AuthenticationHandler()
    authorization = AuthorizationHandler()
    rate_limit = RateLimitHandler(requests_per_minute=2)
    logging = LoggingHandler()
    
    authentication.set_next(authorization).set_next(rate_limit).set_next(logging)
    return authentication

# Test the pipeline
pipeline = create_request_pipeline()

requests = [
    Request("user1", "/api/data", "read"),
    Request("user1", "/api/data", "write"),
    Request("user2", "/api/data", "write"),
    Request("invalid_user", "/api/data", "read"),
    Request("admin", "/api/data", "delete")
]

for request in requests:
    result = pipeline.handle(request)
    if result:
        print(f"Request failed: {result}")
    else:
        print(f"Request successful: {request.user_id} - {request.action}")
```

## 2. Command
Encapsulates requests as objects.

```python
from dataclasses import dataclass
from typing import List

@dataclass
class Order:
    id: str
    quantity: int

class OrderCommand:
    def execute(self):
        pass
    
    def undo(self):
        pass

class AddOrderCommand(OrderCommand):
    def __init__(self, order, orders_list):
        self.order = order
        self.orders = orders_list
    
    def execute(self):
        self.orders.append(self.order)
    
    def undo(self):
        self.orders.remove(self.order)

class OrderProcessor:
    def __init__(self):
        self.orders: List[Order] = []
        self.history: List[OrderCommand] = []
    
    def add_order(self, order):
        cmd = AddOrderCommand(order, self.orders)
        cmd.execute()
        self.history.append(cmd)
    
    def undo_last(self):
        if self.history:
            self.history.pop().undo()

# Usage
processor = OrderProcessor()
processor.add_order(Order("A123", 5))
processor.add_order(Order("B456", 3))
processor.undo_last()  # Removes last order
```

## 3. Mediator
Centralizes complex communications between components.

```python
class ChatMediator:
    def __init__(self):
        self.users = {}
    
    def register(self, user):
        self.users[user.name] = user
    
    def send(self, message, from_user, to_user):
        if to_user in self.users:
            self.users[to_user].receive(message, from_user)

class ChatUser:
    def __init__(self, name, mediator):
        self.name = name
        self.mediator = mediator
        self.messages = []
        mediator.register(self)
    
    def send(self, message, to_user):
        self.mediator.send(message, self.name, to_user)
    
    def receive(self, message, from_user):
        self.messages.append(f"From {from_user}: {message}")

# Usage
chat = ChatMediator()
alice = ChatUser("Alice", chat)
bob = ChatUser("Bob", chat)

alice.send("Hello Bob!", "Bob")
print(bob.messages[-1])  # "From Alice: Hello Bob!"
```

## 4. Memento
Captures and restores object states.

```python
class EditorState:
    def __init__(self, content):
        self.content = content

class Editor:
    def __init__(self):
        self.content = ""
        self.history = []
    
    def write(self, text):
        self.content += text
        self.save_state()
    
    def save_state(self):
        self.history.append(EditorState(self.content))
    
    def undo(self):
        if len(self.history) > 1:
            self.history.pop()  # Remove current
            previous_state = self.history[-1]
            self.content = previous_state.content

# Usage
editor = Editor()
editor.write("Hello")
editor.write(" World")
print(editor.content)  # "Hello World"
editor.undo()
print(editor.content)  # "Hello"
```

## 5. Iterator
Provides sequential access to collection elements.

```python
class TreeNode:
    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None

class BinaryTreeIterator:
    def __init__(self, root):
        self.stack = []
        self._leftmost_inorder(root)
    
    def _leftmost_inorder(self, root):
        while root:
            self.stack.append(root)
            root = root.left
    
    def next(self):
        node = self.stack.pop()
        if node.right:
            self._leftmost_inorder(node.right)
        return node.value
    
    def has_next(self):
        return len(self.stack) > 0

# Usage
root = TreeNode(1)
root.left = TreeNode(2)
root.right = TreeNode(3)

iterator = BinaryTreeIterator(root)
while iterator.has_next():
    print(iterator.next())  # Prints inorder traversal
```

## 6. Observer Pattern
Defines one-to-many dependencies between objects.

```python
from abc import ABC, abstractmethod
from typing import List

class NewsAgency:
    def __init__(self):
        self._subscribers: List['NewsSubscriber'] = []
        self._latest_news = ""

    def attach(self, subscriber: 'NewsSubscriber'):
        self._subscribers.append(subscriber)

    def detach(self, subscriber: 'NewsSubscriber'):
        self._subscribers.remove(subscriber)

    def notify(self):
        for subscriber in self._subscribers:
            subscriber.update(self._latest_news)

    def publish_news(self, news: str):
        self._latest_news = news
        self.notify()

class NewsSubscriber(ABC):
    @abstractmethod
    def update(self, news: str):
        pass

class NewsChannel(NewsSubscriber):
    def __init__(self, name: str):
        self.name = name
        self.news = ""

    def update(self, news: str):
        self.news = news
        print(f"{self.name} received: {self.news}")

# Usage
agency = NewsAgency()
bbc = NewsChannel("BBC")
cnn = NewsChannel("CNN")

agency.attach(bbc)
agency.attach(cnn)
agency.publish_news("Breaking: Major tech announcement!")
```

## 7. State Pattern 
Allows object behavior to change based on internal state.

```python
from abc import ABC, abstractmethod

class VendingState(ABC):
    @abstractmethod
    def insert_coin(self, vending_machine):
        pass

    @abstractmethod
    def eject_coin(self, vending_machine):
        pass

    @abstractmethod
    def dispense(self, vending_machine):
        pass

class NoCoinState(VendingState):
    def insert_coin(self, machine):
        print("Coin accepted")
        machine.state = machine.has_coin_state

    def eject_coin(self, machine):
        print("No coin to eject")

    def dispense(self, machine):
        print("Please insert a coin first")

class HasCoinState(VendingState):
    def insert_coin(self, machine):
        print("Already has a coin")
        
    def eject_coin(self, machine):
        print("Coin returned")
        machine.state = machine.no_coin_state

    def dispense(self, machine):
        print("Item dispensed")
        machine.state = machine.no_coin_state

class VendingMachine:
    def __init__(self):
        self.no_coin_state = NoCoinState()
        self.has_coin_state = HasCoinState()
        self.state = self.no_coin_state

    def insert_coin(self):
        self.state.insert_coin(self)

    def eject_coin(self):
        self.state.eject_coin(self)

    def dispense(self):
        self.state.dispense(self)

# Usage
machine = VendingMachine()
machine.insert_coin()
machine.dispense()
```

## 8. Strategy Pattern
Makes algorithms interchangeable within a family.

```python
from abc import ABC, abstractmethod
from typing import List

class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount: float) -> bool:
        pass

class CreditCardPayment(PaymentStrategy):
    def __init__(self, card_number: str, expiry: str):
        self.card_number = card_number
        self.expiry = expiry

    def pay(self, amount: float) -> bool:
        print(f"Paid ${amount} using Credit Card {self.card_number}")
        return True

class PayPalPayment(PaymentStrategy):
    def __init__(self, email: str):
        self.email = email

    def pay(self, amount: float) -> bool:
        print(f"Paid ${amount} using PayPal account {self.email}")
        return True

class ShoppingCart:
    def __init__(self):
        self.items: List[float] = []
        self.payment_strategy: PaymentStrategy = None

    def add_item(self, price: float):
        self.items.append(price)

    def set_payment_strategy(self, strategy: PaymentStrategy):
        self.payment_strategy = strategy

    def checkout(self) -> bool:
        amount = sum(self.items)
        return self.payment_strategy.pay(amount)

# Usage
cart = ShoppingCart()
cart.add_item(100)
cart.add_item(50)
cart.set_payment_strategy(CreditCardPayment("1234-5678", "12/24"))
cart.checkout()
```

## 9. Template Method Pattern
Defines algorithm skeleton, lets subclasses override specific steps.

```python
from abc import ABC, abstractmethod

class DataMiner(ABC):
    def mine(self, path: str):
        file = self.open_file(path)
        data = self.extract_data(file)
        self.parse_data(data)
        self.analyze_data()
        self.send_report()
        self.close_file(file)

    @abstractmethod
    def open_file(self, path: str):
        pass

    @abstractmethod
    def extract_data(self, file):
        pass

    def analyze_data(self):
        print("Analyzing data...")

    def send_report(self):
        print("Sending report...")

    def close_file(self, file):
        print("Closing file...")

class PDFDataMiner(DataMiner):
    def open_file(self, path: str):
        print(f"Opening PDF file: {path}")
        return "pdf_file"

    def extract_data(self, file):
        print("Extracting data from PDF")
        return "pdf_data"

class DocxDataMiner(DataMiner):
    def open_file(self, path: str):
        print(f"Opening DOCX file: {path}")
        return "docx_file"

    def extract_data(self, file):
        print("Extracting data from DOCX")
        return "docx_data"

# Usage
pdf_miner = PDFDataMiner()
pdf_miner.mine("document.pdf")
```

## 10. Visitor Pattern
Separates algorithms from objects they operate on.

```python
from abc import ABC, abstractmethod
from typing import List

class DocumentPart(ABC):
    @abstractmethod
    def accept(self, visitor: 'DocumentVisitor'):
        pass

class Heading(DocumentPart):
    def __init__(self, text: str):
        self.text = text

    def accept(self, visitor: 'DocumentVisitor'):
        return visitor.visit_heading(self)

class Paragraph(DocumentPart):
    def __init__(self, text: str):
        self.text = text

    def accept(self, visitor: 'DocumentVisitor'):
        return visitor.visit_paragraph(self)

class DocumentVisitor(ABC):
    @abstractmethod
    def visit_heading(self, heading: Heading):
        pass

    @abstractmethod
    def visit_paragraph(self, paragraph: Paragraph):
        pass

class HTMLVisitor(DocumentVisitor):
    def visit_heading(self, heading: Heading):
        return f"<h1>{heading.text}</h1>"

    def visit_paragraph(self, paragraph: Paragraph):
        return f"<p>{paragraph.text}</p>"

class MarkdownVisitor(DocumentVisitor):
    def visit_heading(self, heading: Heading):
        return f"# {heading.text}"

    def visit_paragraph(self, paragraph: Paragraph):
        return paragraph.text

# Usage
document = [
    Heading("Title"),
    Paragraph("Content")
]

html_visitor = HTMLVisitor()
markdown_visitor = MarkdownVisitor()

for part in document:
    print(part.accept(html_visitor))
    print(part.accept(markdown_visitor))
```

## Pattern Combinations
## Before Refactoring
```python
# 1. Content Management System - Before
class Document:
    def __init__(self):
        self.elements = []
    
    def add_element(self, element):
        self.elements.append(element)
        
class Paragraph:
    def __init__(self, text):
        self.text = text

# Usage
doc = Document()
doc.add_element(Paragraph("Hello"))

# 2. Payment Processing - Before
def process_payment(method, amount):
    if method == "paypal":
        print(f"Processing ${amount} via PayPal")
    elif method == "credit":
        print(f"Processing ${amount} via Credit Card")

# Usage
process_payment("paypal", 100)

# 3. Game Engine - Before
class GameObject:
    def __init__(self):
        self.state = "idle"
    
    def update(self):
        if self.state == "idle":
            print("Idle animation")

# Usage
player = GameObject()
player.update()

# 4. Caching Service - Before
cache = {}

def get_data(key):
    if key in cache:
        return cache[key]
    data = f"Data for {key}"  # Simulate fetch
    cache[key] = data
    return data

# Usage
data = get_data("user_1")

# 5. IoT Device Control - Before
class SmartHome:
    def process_command(self, command):
        if command == "turn_on_lights":
            print("Lights on")
        elif command == "set_temperature":
            print("Temperature set")
        elif command == "activate_security":
            print("Security activated")

# Usage
home = SmartHome()
home.process_command("turn_on_lights")
```
## After Refactoring
## Content Management System
```python
# Combines: Composite + Command + Observer
class Document:
    def __init__(self):
        self.elements = []
        self.observers = []
        self.command_history = []

    def add_element(self, element):
        cmd = AddElementCommand(self, element)
        cmd.execute()
        self.command_history.append(cmd)
        self.notify_observers()

class DocumentElement:
    def render(self): pass

class Paragraph(DocumentElement):
    def __init__(self, text):
        self.text = text

class AddElementCommand:
    def __init__(self, document, element):
        self.document = document
        self.element = element

    def execute(self):
        self.document.elements.append(self.element)

    def undo(self):
        self.document.elements.remove(self.element)

# Usage
doc = Document()
doc.add_element(Paragraph("Hello"))
```

## 2. Payment Processing System
```python
# Combines: Strategy + Template Method + Factory
class PaymentProcessor:
    def process_payment(self, amount):
        self.validate()
        self.process(amount)
        self.notify_completion()

    def validate(self): pass
    def process(self, amount): pass
    def notify_completion(self): pass

class PayPalProcessor(PaymentProcessor):
    def __init__(self, email):
        self.email = email

    def process(self, amount):
        print(f"Processing ${amount} via PayPal to {self.email}")

class PaymentFactory:
    @staticmethod
    def create_processor(method, **kwargs):
        if method == "paypal":
            return PayPalProcessor(kwargs.get("email"))
        # Add other payment methods

# Usage
processor = PaymentFactory.create_processor("paypal", email="user@example.com")
processor.process_payment(100)
```

## 3. Game Engine Components
```python
# Combines: State + Observer + Pool
class GameObject:
    def __init__(self):
        self.state = None
        self.observers = []

    def set_state(self, state):
        self.state = state
        self.notify_observers()

    def add_observer(self, observer):
        self.observers.append(observer)

class GameObjectPool:
    def __init__(self):
        self.available_objects = []
        self.max_objects = 10

    def acquire(self):
        if not self.available_objects:
            if len(self.available_objects) < self.max_objects:
                return GameObject()
        return self.available_objects.pop()

    def release(self, obj):
        self.available_objects.append(obj)

# Usage
pool = GameObjectPool()
player = pool.acquire()
```

## 4. Caching Service
```python
# Combines: Proxy + Flyweight + Singleton
class CacheManager:
    _instance = None

    def __new__(cls):
        if not cls._instance:
            cls._instance = super().__new__(cls)
            cls._instance.cache = {}
            cls._instance.data_pool = {}
        return cls._instance

    def get_data(self, key):
        if key in self.cache:
            return self.cache[key]
        data = self.fetch_data(key)
        self.cache[key] = data
        return data

    def fetch_data(self, key):
        # Simulate expensive data fetch
        return f"Data for {key}"

# Usage
cache = CacheManager()
data = cache.get_data("user_1")
```

## 5. IoT Device Management
```python
# Combines: Facade + Mediator + Chain of Responsibility
class SmartHomeFacade:
    def __init__(self):
        self.mediator = DeviceMediator()
        self.handler_chain = SecurityHandler(
            TemperatureHandler(
                LightingHandler(None)
            )
        )

    def process_command(self, command):
        self.handler_chain.handle(command)
        self.mediator.notify_all(command)

class DeviceMediator:
    def __init__(self):
        self.devices = []

    def notify_all(self, command):
        for device in self.devices:
            device.receive(command)

class CommandHandler:
    def __init__(self, next_handler):
        self.next = next_handler

    def handle(self, command):
        if self.next:
            self.next.handle(command)

# Usage
smart_home = SmartHomeFacade()
smart_home.process_command("turn_on_lights")
```

## Conclusion
The Gang of Four (GoF) design patterns offer timeless, reusable solutions to common software design challenges. By categorizing patterns into **Creational**, **Structural**, and **Behavioral**, developers gain a structured approach to solving complex design issues with scalable and maintainable code. Adopting these patterns fosters better collaboration, flexibility, and efficiency in software development. Understanding these principles deeply can help in choosing the right pattern for the right problem, resulting in cleaner and more robust systems.

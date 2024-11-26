# Understanding Controller and Operator Patterns in Python

## Table of Contents
1. [Introduction](#introduction)
2. [Controller Pattern](#controller-pattern)
   - [Basic Concepts](#basic-concepts)
   - [Example: Smart Home](#example-smart-home)
   - [Advanced Example: Traffic Management](#advanced-example-traffic-management)
3. [Operator Pattern](#operator-pattern)
   - [Core Concepts](#core-concepts)
   - [Example: Coffee Shop Chain](#example-coffee-shop-chain)
   - [Example: Database Management](#example-database-management)
4. [Best Practices](#best-practices)
5. [Conclusion](#conclusion)

## Introduction

In Python, the Controller and Operator patterns can be implemented with a focus on readability and simplicity, following the "explicit is better than implicit" philosophy. These patterns are particularly useful in automation and system management.

## Controller Pattern

### Basic Concepts

Here's a simple implementation showcasing Python's approach to the controller pattern:

```python
from dataclasses import dataclass
from time import sleep
from typing import Protocol

class Controller(Protocol):
    def reconcile(self) -> None:
        """Reconcile actual state with desired state."""
        pass

@dataclass
class SimpleController:
    desired: int
    actual: int

    def reconcile(self) -> None:
        if self.actual < self.desired:
            self.actual += 1
            print(f"⬆️ Increasing value to {self.actual}")
        elif self.actual > self.desired:
            self.actual -= 1
            print(f"⬇️ Decreasing value to {self.actual}")
        else:
            print("✓ Values matched")

def main():
    controller = SimpleController(desired=5, actual=2)
    
    for step in range(5):
        print(f"\nStep {step + 1}: Actual={controller.actual}, Desired={controller.desired}")
        controller.reconcile()
        sleep(1)

if __name__ == "__main__":
    main()
```

### Example: Smart Home

A more practical example using Python's object-oriented features:

```python
from dataclasses import dataclass
from typing import List
import asyncio

@dataclass
class Room:
    name: str
    actual_temp: float
    desired_temp: float
    humidity: float

class SmartHomeController:
    def __init__(self, rooms: List[Room], power_saving: bool = True):
        self.rooms = rooms
        self.power_saving = power_saving

    async def reconcile(self) -> None:
        """Asynchronously reconcile temperature for all rooms."""
        await asyncio.gather(*[
            self._adjust_room_temperature(room)
            for room in self.rooms
        ])

    async def _adjust_room_temperature(self, room: Room) -> None:
        """Adjust temperature for a single room."""
        temp_diff = room.actual_temp - room.desired_temp
        
        if abs(temp_diff) < 0.5:
            print(f"✓ {room.name}: Temperature optimal ({room.actual_temp:.1f}°C)")
            return

        if temp_diff < 0:
            print(f"🔥 {room.name}: Heating ({room.actual_temp:.1f}°C → {room.desired_temp:.1f}°C)")
            room.actual_temp += 0.5
        else:
            print(f"❄️ {room.name}: Cooling ({room.actual_temp:.1f}°C → {room.desired_temp:.1f}°C)")
            room.actual_temp -= 0.5

async def main():
    controller = SmartHomeController([
        Room("Living Room", actual_temp=18.5, desired_temp=21.0, humidity=45),
        Room("Bedroom", actual_temp=23.0, desired_temp=20.0, humidity=50),
    ])
    
    for cycle in range(5):
        print(f"\n=== Cycle {cycle + 1} ===")
        await controller.reconcile()
        await asyncio.sleep(1)

if __name__ == "__main__":
    asyncio.run(main())
```

### Example: Traffic Management

Using more advanced Python features for complex state management:

```python
from dataclasses import dataclass
from enum import Enum
from typing import Dict
import asyncio

class LightStatus(Enum):
    RED = "red"
    YELLOW = "yellow"
    GREEN = "green"

@dataclass
class Intersection:
    id: str
    current_flow: int  # vehicles per minute
    desired_flow: int
    light_status: LightStatus
    queue_length: int

    @property
    def is_congested(self) -> bool:
        return self.queue_length > 10

class TrafficController:
    def __init__(self, intersections: Dict[str, Intersection]):
        self.intersections = intersections
        self.peak_hours = False

    async def reconcile(self) -> None:
        """Reconcile traffic flow across all intersections."""
        tasks = [
            self._manage_intersection(intersection)
            for intersection in self.intersections.values()
        ]
        await asyncio.gather(*tasks)

    async def _manage_intersection(self, intersection: Intersection) -> None:
        """Manage traffic flow for a single intersection."""
        if intersection.is_congested:
            print(f"⚠️ Heavy traffic at intersection {intersection.id}")
            await self._adjust_traffic_flow(intersection)

        if intersection.current_flow < intersection.desired_flow:
            print(f"🚦 Adjusting light timing at {intersection.id}")
            intersection.light_status = LightStatus.GREEN
            intersection.current_flow += 2

        print(f"Intersection {intersection.id}: "
              f"Flow={intersection.current_flow}/min, "
              f"Queue={intersection.queue_length}")

    async def _adjust_traffic_flow(self, intersection: Intersection) -> None:
        """Adjust traffic flow parameters."""
        intersection.queue_length = max(0, intersection.queue_length - 5)
        intersection.current_flow += 3
```

## Operator Pattern

### Example: Coffee Shop Chain

Implementing the operator pattern with Python's type hints and async features:

```python
from dataclasses import dataclass
from typing import Dict, Optional
from enum import Enum
import asyncio

class ShopStatus(Enum):
    OPEN = "open"
    CLOSED = "closed"
    MAINTENANCE = "maintenance"

@dataclass
class CoffeeShop:
    location: str
    inventory: Dict[str, int]
    staff_count: int
    queue_length: int
    status: ShopStatus

class CoffeeChainOperator:
    def __init__(
        self,
        shops: Dict[str, CoffeeShop],
        restock_levels: Dict[str, int]
    ):
        self.shops = shops
        self.restock_levels = restock_levels
        self._peak_hours = False

    @property
    def peak_hours(self) -> bool:
        return self._peak_hours

    @peak_hours.setter
    def peak_hours(self, value: bool) -> None:
        self._peak_hours = value
        print(f"Peak hours {'enabled' if value else 'disabled'}")

    async def reconcile(self) -> None:
        """Reconcile all coffee shop operations."""
        tasks = [
            self._manage_shop(shop)
            for shop in self.shops.values()
        ]
        await asyncio.gather(*tasks)

    async def _manage_shop(self, shop: CoffeeShop) -> None:
        """Manage operations for a single coffee shop."""
        await self._check_inventory(shop)
        await self._manage_staffing(shop)
        await self._monitor_performance(shop)

    async def _check_inventory(self, shop: CoffeeShop) -> None:
        """Check and update inventory levels."""
        for item, count in shop.inventory.items():
            if count < self.restock_levels[item]:
                print(f"📦 Ordering more {item} for {shop.location}")
                shop.inventory[item] += 50

    async def _manage_staffing(self, shop: CoffeeShop) -> None:
        """Adjust staffing based on queue length."""
        if shop.queue_length > 10 and shop.staff_count < 4:
            print(f"👥 Adding staff at {shop.location}")
            shop.staff_count += 1

    async def _monitor_performance(self, shop: CoffeeShop) -> None:
        """Monitor and report shop performance."""
        print(f"{shop.location}: Queue={shop.queue_length}, "
              f"Staff={shop.staff_count}")
```

## Best Practices

1. **Use Type Hints**
   - Leverage Python's type system for better code clarity
   - Use protocols for interface definitions
   - Define clear data structures with dataclasses

2. **Async Operations**
   - Use asyncio for concurrent operations
   - Implement proper error handling with try/except
   - Use async context managers when appropriate

3. **Python Idioms**
   - Follow PEP 8 style guidelines
   - Use properties for computed attributes
   - Implement proper dunders when needed

4. **Error Handling**
```python
from typing import Optional

class ControllerError(Exception):
    """Base exception for controller operations."""
    pass

class ReconciliationError(ControllerError):
    """Raised when reconciliation fails."""
    pass

async def safe_reconcile(controller: Controller) -> Optional[str]:
    """Safely reconcile state with error handling."""
    try:
        await controller.reconcile()
        return None
    except ReconciliationError as e:
        print(f"Reconciliation failed: {e}")
        return str(e)
    except Exception as e:
        print(f"Unexpected error: {e}")
        raise
```

## Conclusion

The Python implementation of Controller and Operator patterns emphasizes:

1. **Readability**
   - Clear class hierarchies
   - Type hints for better understanding
   - Descriptive naming

2. **Modularity**
   - Separate concerns into classes
   - Use async functions for concurrent operations
   - Define clear interfaces

3. **Robustness**
   - Proper error handling
   - State validation
   - Clear logging and monitoring

Python's features make these patterns particularly effective:
- Type hints improve code clarity
- Async support enables efficient operations
- Dataclasses reduce boilerplate
- Properties provide clean interfaces
- Protocol classes define clear contracts

These patterns in Python provide a powerful way to manage complex systems while maintaining code readability and maintainability.

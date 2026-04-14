# Dataclasses

## Описание

`dataclasses` (Python 3.7+) — модуль для автоматической генерации dunder методов (`__init__`, `__repr__`, `__eq__` и др.) для классов данных.

## Базовое использование

```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    email: str
    age: int

# Автоматически создаются:
# __init__(self, name, email, age)
# __repr__(self)
# __eq__(self, other)

user = User("Alice", "alice@example.com", 30)
print(user)  # User(name='Alice', email='alice@example.com', age=30)
print(user.name)  # 'Alice'
```

## Значения по умолчанию

```python
from dataclasses import dataclass

@dataclass
class Product:
    name: str
    price: float
    quantity: int = 0
    category: str = "General"

product = Product("Laptop", 999.99)
print(product.quantity)  # 0
print(product.category)  # 'General'

# Порядок: поля без дефолтов → с дефолтами
```

## field() — Тонкая настройка

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Team:
    name: str
    members: List[str] = field(default_factory=list)
    # mutable default factory

    # Поле не будет в __init__
    _internal_id: int = field(default=0, init=False)

    # Поле не будет в __repr__
    secret: str = field(default="hidden", repr=False)

    # Поле не будет в сравнении
    temp: float = field(default=0.0, compare=False)

team = Team("Dev")
team.members.append("Alice")
print(team)  # Team(name='Dev', members=['Alice'], secret=...)
```

## frozen — Immutable dataclass

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Point:
    x: float
    y: float

point = Point(1.0, 2.0)
# point.x = 3.0  # FrozenInstanceError!

# Можно через object.__setattr__
object.__setattr__(point, 'x', 3.0)  # Не рекомендуется
```

## Методы в dataclass

```python
from dataclasses import dataclass

@dataclass
class Rectangle:
    width: float
    height: float

    def area(self) -> float:
        return self.width * self.height

    def perimeter(self) -> float:
        return 2 * (self.width + self.height)

    def is_square(self) -> bool:
        return self.width == self.height

rect = Rectangle(5, 10)
print(rect.area())      # 50.0
print(rect.perimeter()) # 30.0
```

## Наследование

```python
from dataclasses import dataclass

@dataclass
class Person:
    name: str
    age: int

@dataclass
class Employee(Person):
    employee_id: int
    department: str

emp = Employee("Alice", 30, 1234, "Engineering")
print(emp)  # Employee(name='Alice', age=30, employee_id=1234, department='Engineering')
```

## post_init — Дополнительная инициализация

```python
from dataclasses import dataclass, field

@dataclass
class Circle:
    radius: float
    area: float = field(init=False)

    def __post_init__(self):
        # Вызывается после __init__
        self.area = 3.14159 * self.radius ** 2

circle = Circle(5)
print(circle.area)  # 78.53975
```

## asdict и astuple

```python
from dataclasses import dataclass, asdict, astuple

@dataclass
class Point:
    x: int
    y: int

point = Point(3, 4)

# В dict/tuple
print(asdict(point))    # {'x': 3, 'y': 4}
print(astuple(point))   # (3, 4)

# Вложенные структуры
@dataclass
class Address:
    city: str
    country: str

@dataclass
class User:
    name: str
    address: Address

user = User("Alice", Address("Moscow", "Russia"))
print(asdict(user))
# {'name': 'Alice', 'address': {'city': 'Moscow', 'country': 'Russia'}}
```

## Сравнение с attrs

```python
# attrs — более мощная альтернатива
# pip install attrs

from attrs import define, field

@define
class Product:
    name: str
    price: float
    quantity: int = field(default=0)

    @price.validator
    def check_price(self, attribute, value):
        if value < 0:
            raise ValueError("Price must be non-negative")
```

## Best Practices

✅ **Используйте** `field(default_factory=...)` для mutable дефолтов
✅ **Используйте** `frozen=True` для immutable объектов
✅ **Используйте** `repr=False` для секретных полей
✅ **Используйте** `post_init` для вычисляемых полей

❌ **Не используйте** mutable дефолты (`members: list = []`)
❌ **Не храните** сложную логику в dataclass (это DTO)
❌ **Не путайте** dataclass с namedtuple (разные цели)

## Ссылки

- [Официальная документация dataclasses](https://docs.python.org/3/library/dataclasses.html)
- [PEP 557 — Data Classes](https://peps.python.org/pep-0557/)
- [attrs — расширенная альтернатива](https://www.attrs.org/)

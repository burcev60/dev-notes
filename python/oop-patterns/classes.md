# Классы и наследование в Python

## Описание

Python поддерживает объектно-ориентированное программирование с классами, наследованием, миксинами и специальными методами.

## Базовый класс

```python
class User:
    # Конструктор
    def __init__(self, name: str, email: str) -> None:
        self.name = name
        self.email = email

    # Метод экземпляра
    def greet(self) -> str:
        return f"Hello, {self.name}!"

    # Строковое представление
    def __str__(self) -> str:
        return f"User({self.name}, {self.email})"

    def __repr__(self) -> str:
        return f"User(name={self.name!r}, email={self.email!r})"

# Создание
user = User("Alice", "alice@example.com")
print(user.greet())  # Hello, Alice!
```

## Атрибуты класса vs экземпляра

```python
class Counter:
    # Атрибут класса (общий для всех)
    total = 0

    def __init__(self, name: str) -> None:
        # Атрибут экземпляра
        self.name = name
        self.count = 0

    def increment(self) -> None:
        self.count += 1
        Counter.total += 1

c1 = Counter("A")
c2 = Counter("B")
c1.increment()
c2.increment()
print(c1.count, c2.count, Counter.total)  # 1 1 2
```

## Свойства (property)

```python
class Circle:
    def __init__(self, radius: float) -> None:
        self.radius = radius

    @property
    def area(self) -> float:
        return 3.14159 * self.radius ** 2

    @property
    def diameter(self) -> float:
        return 2 * self.radius

    @diameter.setter
    def diameter(self, value: float) -> None:
        self.radius = value / 2

circle = Circle(5)
print(circle.area)       # 78.53975
print(circle.diameter)   # 10.0
circle.diameter = 20.0
print(circle.radius)     # 10.0
```

## Приватные атрибуты

```python
class BankAccount:
    def __init__(self, balance: float) -> None:
        self._balance = balance  # "Protected" (соглашение)
        self.__pin = 1234        # "Private" (name mangling)

    def get_balance(self) -> float:
        return self._balance

    def __validate_pin(self, pin: int) -> bool:
        return pin == self.__pin

account = BankAccount(1000)
account._balance         # 1000 (доступно, но не рекомендуется)
# account.__pin          # AttributeError
account._BankAccount__pin  # 1234 (name mangling)
```

## Наследование

```python
class Animal:
    def __init__(self, name: str) -> None:
        self.name = name

    def speak(self) -> str:
        raise NotImplementedError

    def __str__(self) -> str:
        return f"Animal({self.name})"

class Dog(Animal):
    def __init__(self, name: str, breed: str) -> None:
        super().__init__(name)  # Вызов родительского конструктора
        self.breed = breed

    def speak(self) -> str:
        return "Woof!"

class Cat(Animal):
    def speak(self) -> str:
        return "Meow!"

dog = Dog("Rex", "Labrador")
cat = Cat("Whiskers")
print(dog.speak())  # Woof!
print(cat.speak())  # Meow!
```

## Множественное наследование

```python
class Flyer:
    def fly(self) -> str:
        return "Flying..."

class Swimmer:
    def swim(self) -> str:
        return "Swimming..."

class Duck(Flyer, Swimmer):
    def __init__(self, name: str) -> None:
        self.name = name

duck = Duck("Donald")
print(duck.fly())   # Flying...
print(duck.swim())  # Swimming...

# MRO — Method Resolution Order
print(Duck.__mro__)  # (Duck, Flyer, Swimmer, object)
```

## Миксины

```python
class JsonMixin:
    def to_json(self) -> str:
        import json
        return json.dumps(self.__dict__)

class LogMixin:
    def log(self, message: str) -> None:
        print(f"[{self.__class__.__name__}] {message}")

class Service(JsonMixin, LogMixin):
    def __init__(self, name: str) -> None:
        self.name = name
        self.status = "running"

service = Service("API")
service.log("Started")          # [Service] Started
print(service.to_json())        # {"name": "API", "status": "running"}
```

## Специальные методы (dunder)

```python
class Vector:
    def __init__(self, x: int, y: int) -> None:
        self.x = x
        self.y = y

    # Арифметика
    def __add__(self, other: 'Vector') -> 'Vector':
        return Vector(self.x + other.x, self.y + other.y)

    def __mul__(self, scalar: int) -> 'Vector':
        return Vector(self.x * scalar, self.y * scalar)

    # Сравнение
    def __eq__(self, other: object) -> bool:
        if not isinstance(other, Vector):
            return NotImplemented
        return self.x == other.x and self.y == other.y

    def __lt__(self, other: 'Vector') -> bool:
        return (self.x ** 2 + self.y ** 2) < (other.x ** 2 + other.y ** 2)

    # Представление
    def __repr__(self) -> str:
        return f"Vector({self.x}, {self.y})"

    def __str__(self) -> str:
        return f"({self.x}, {self.y})"

    # Длина
    def __len__(self) -> int:
        return int((self.x ** 2 + self.y ** 2) ** 0.5)

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)     # Vector(4, 6)
print(v1 * 3)      # Vector(3, 6)
print(v1 == v2)    # False
print(len(v1))     # 2
```

## Best Practices

✅ **Используйте** `super()` для вызова методов родителя
✅ **Используйте** `@property` для вычисляемых атрибутов
✅ **Используйте** миксины для переиспользуемого поведения
✅ **Реализуйте** `__repr__` для отладки

❌ **Не используйте** множественное наследование без необходимости
❌ **Не злоупотребляйте** приватными атрибутами (`__name`)
❌ **Не забывайте** `NotImplemented` в методах сравнения

## Ссылки

- [Официальная документация classes](https://docs.python.org/3/tutorial/classes.html)
- [Data model — dunder методы](https://docs.python.org/3/reference/datamodel.html)

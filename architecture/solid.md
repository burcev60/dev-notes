# SOLID принципы

## Описание

SOLID — 5 принципов объектно-ориентированного проектирования.

## S — Single Responsibility Principle

Каждый класс должен иметь одну зону ответственности.

```python
# ❌ Нарушение
class User:
    def save_to_db(self): ...
    def send_email(self): ...
    def generate_report(self): ...

# ✅ Правильно
class User:
    def __init__(self, name: str): ...

class UserRepository:
    def save(self, user: User): ...

class EmailService:
    def send(self, user: User): ...
```

## O — Open/Closed Principle

Открыты для расширения, закрыты для модификации.

```python
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    @abstractmethod
    def process(self, amount: float) -> None: ...

class StripeProcessor(PaymentProcessor):
    def process(self, amount: float) -> None: ...

class PayPalProcessor(PaymentProcessor):
    def process(self, amount: float) -> None: ...

# Добавление нового способа без изменения существующего кода
```

## L — Liskov Substitution Principle

Подклассы должны заменять базовые классы.

```python
class Bird:
    def fly(self): ...

class Sparrow(Bird):
    def fly(self): ...  # OK

class Penguin(Bird):
    def fly(self):
        raise Exception("Can't fly")  # Нарушение!

# Правильно
class Bird: ...
class FlyingBird(Bird):
    def fly(self): ...
class Penguin(Bird): ...  # Нет fly()
```

## I — Interface Segregation Principle

Много специфических интерфейсов лучше одного универсального.

```python
# ❌
class Worker(ABC):
    @abstractmethod
    def work(self): ...
    @abstractmethod
    def eat(self): ...

# ✅
class Workable(ABC):
    @abstractmethod
    def work(self): ...

class Eatable(ABC):
    @abstractmethod
    def eat(self): ...
```

## D — Dependency Inversion Principle

Зависимость от абстракций, а не от реализаций.

```python
# ❌
class Service:
    def __init__(self):
        self.db = MySQLDatabase()

# ✅
class Service:
    def __init__(self, db: DatabaseInterface):
        self.db = db
```

## Best Practices

✅ **Применяйте** SOLID для масштабируемого кода
✅ **Не фанатейте** — иногда проще нарушить принцип

## Ссылки

- [SOLID Wikipedia](https://en.wikipedia.org/wiki/SOLID)

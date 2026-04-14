# Design Patterns — Порождающие

## Описание

Порождающие паттерны отвечают за создание объектов.

## Factory Method

```python
from abc import ABC, abstractmethod

class Notification(ABC):
    @abstractmethod
    def send(self, message: str) -> None: ...

class EmailNotification(Notification):
    def send(self, message: str) -> None: ...

class SMSNotification(Notification):
    def send(self, message: str) -> None: ...

class NotificationFactory:
    @staticmethod
    def create(type: str) -> Notification:
        if type == "email":
            return EmailNotification()
        elif type == "sms":
            return SMSNotification()
        raise ValueError(f"Unknown type: {type}")

# Использование
notifier = NotificationFactory.create("email")
notifier.send("Hello!")
```

## Builder

```python
class QueryBuilder:
    def __init__(self):
        self._table = None
        self._conditions = []

    def table(self, name: str) -> 'QueryBuilder':
        self._table = name
        return self

    def where(self, condition: str) -> 'QueryBuilder':
        self._conditions.append(condition)
        return self

    def build(self) -> str:
        query = f"SELECT * FROM {self._table}"
        if self._conditions:
            query += " WHERE " + " AND ".join(self._conditions)
        return query

sql = QueryBuilder().table("users").where("age > 18").build()
# SELECT * FROM users WHERE age > 18
```

## Singleton

```python
class Singleton:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

# Pythonic way
class Settings:
    database_url: str = "postgresql://localhost/db"
    secret_key: str = "changeme"

# Использование
settings = Settings()
```

## Best Practices

✅ **Используйте** Factory для создания разных типов
✅ **Используйте** Builder для сложных объектов
✅ **Избегайте** Singleton — это часто anti-pattern

## Ссылки

- [Refactoring Guru — Factory](https://refactoring.guru/design-patterns/factory-method)
- [Refactoring Guru — Builder](https://refactoring.guru/design-patterns/builder)

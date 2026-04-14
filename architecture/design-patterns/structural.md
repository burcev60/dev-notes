# Design Patterns — Структурные

## Описание

Структурные паттерны отвечают за композицию классов и объектов.

## Adapter

```python
class EuropeanSocket:
    def plug_in(self):
        return 220

class USASocket:
    def plug_in(self):
        return 110

class Adapter(EuropeanSocket):
    def __init__(self, usa_socket: USASocket):
        self.usa_socket = usa_socket

    def plug_in(self):
        return self.usa_socket.plug_in() * 2  # Конвертация
```

## Decorator

```python
class DataSource:
    def read(self) -> str: ...

class FileDataSource(DataSource):
    def read(self) -> str:
        return "file content"

class CompressionDecorator(DataSource):
    def __init__(self, source: DataSource):
        self.source = source

    def read(self) -> str:
        data = self.source.read()
        return compress(data)

# Использование
source = CompressionDecorator(FileDataSource())
data = source.read()
```

## Facade

```python
class AuthFacade:
    def __init__(self):
        self.auth_service = AuthService()
        self.session_service = SessionService()
        self.notification_service = NotificationService()

    def login(self, email: str, password: str) -> Session:
        user = self.auth_service.authenticate(email, password)
        session = self.session_service.create(user)
        self.notification_service.send_welcome_email(user)
        return session

# Простой интерфейс для сложной системы
facade = AuthFacade()
session = facade.login("alice@example.com", "password")
```

## Best Practices

✅ **Используйте** Adapter для интеграции несовместимых API
✅ **Используйте** Facade для упрощения сложных систем
✅ **Используйте** Decorator для добавления поведения

## Ссылки

- [Refactoring Guru — Structural Patterns](https://refactoring.guru/design-patterns/structural-patterns)

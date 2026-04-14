# Декораторы в Python

## Описание

Декораторы — это функции, которые принимают другую функцию и возвращают обёртку, модифицируя её поведение.

## Базовый декоратор

```python
from functools import wraps
from typing import Callable

# Простой декоратор
def logger(func: Callable) -> Callable:
    @wraps(func)  # Сохраняет имя и docstring
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Done {func.__name__}")
        return result
    return wrapper

@logger
def greet(name: str) -> str:
    """Greet someone"""
    return f"Hello, {name}!"

greet("Alice")
# Calling greet
# Done greet
print(greet.__name__)  # 'greet' (благодаря @wraps)
```

## Декоратор с аргументами

```python
from functools import wraps
from typing import Callable

def repeat(times: int) -> Callable:
    """Декоратор с параметром"""
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def say_hello():
    print("Hello!")

say_hello()
# Hello!
# Hello!
# Hello!
```

## Практические примеры

```python
import time
from functools import wraps

# Замер времени
def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start
        print(f"{func.__name__} took {elapsed:.3f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)

# Retry декоратор
import time

def retry(max_attempts: int = 3, delay: float = 1.0):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed: {e}")
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(max_attempts=3, delay=0.5)
def unstable_api_call():
    import random
    if random.random() < 0.7:
        raise ConnectionError("Failed")
    return "Success"

# Авторизация
def require_auth(role: str):
    def decorator(func):
        @wraps(func)
        def wrapper(user, *args, **kwargs):
            if user.role != role:
                raise PermissionError(f"Requires {role} role")
            return func(user, *args, **kwargs)
        return wrapper
    return decorator
```

## Декоратор для методов класса

```python
from functools import wraps

def singleton(cls):
    """Декоратор синглтона"""
    instances = {}

    @wraps(cls)
    def wrapper(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]

    return wrapper

@singleton
class DatabaseConnection:
    def __init__(self, dsn: str) -> None:
        self.dsn = dsn
        print(f"Connected to {dsn}")

db1 = DatabaseConnection("postgres://localhost/db")
db2 = DatabaseConnection("postgres://localhost/db")
print(db1 is db2)  # True (одинаковый экземпляр)
```

## Декоратор с сохранением состояния

```python
from functools import wraps

def counter(func):
    """Считает количество вызовов"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        wrapper.count += 1
        print(f"Call #{wrapper.count}")
        return func(*args, **kwargs)
    wrapper.count = 0
    return wrapper

@counter
def do_something():
    pass

do_something()  # Call #1
do_something()  # Call #2
print(do_something.count)  # 2
```

## Класс как декоратор

```python
from functools import wraps

class Repeater:
    def __init__(self, times: int) -> None:
        self.times = times

    def __call__(self, func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(self.times):
                result = func(*args, **kwargs)
            return result
        return wrapper

@Repeater(3)
def greet():
    print("Hello!")

greet()
```

## Best Practices

✅ **Всегда используйте** `@wraps(func)` для сохранения метаданных
✅ **Обрабатывайте** исключения внутри wrapper
✅ **Используйте** `functools.wraps` для `__name__`, `__doc__`
✅ **Документируйте** поведение декоратора

❌ **Не используйте** декораторы с побочными эффектами
❌ **Не скрывайте** исключения без веской причины
❌ **Не забывайте** передавать `*args, **kwargs`

## Ссылки

- [Официальная документация decorators](https://docs.python.org/3/glossary.html#term-decorator)
- [PEP 318 — Decorators](https://peps.python.org/pep-0318/)

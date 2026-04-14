# functools — Функции высшего порядка

## Описание

Модуль `functools` предоставляет инструменты для функционального программирования: декораторы, частичное применение, кэширование.

## lru_cache — Кэширование

```python
from functools import lru_cache

# Базовое использование
@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# Статистика кэша
print(fibonacci.cache_info())
# CacheInfo(hits=8, misses=10, maxsize=128, currsize=10)

# Очистка кэша
fibonacci.cache_clear()

# Без лимита (осторожно!)
@lru_cache(maxsize=None)
def expensive_computation(x):
    return x ** 2

# Кэширование API запросов
import requests

@lru_cache(typed=True)
def get_user(user_id: int):
    return requests.get(f'https://api.example.com/users/{user_id}').json()
```

> 💡 `typed=True` — кэширует отдельно для разных типов аргументов: `f(1)` и `f(1.0)`.

## cache — Бесконечный кэш (Python 3.9+)

```python
from functools import cache

@cache
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)

# Эквивалентно @lru_cache(maxsize=None)
```

## partial — Частичное применение

```python
from functools import partial

# Фиксация аргументов
def power(base, exp):
    return base ** exp

square = partial(power, exp=2)
cube = partial(power, exp=3)

print(square(5))   # 25
print(cube(3))     # 27

# С позиционными аргументами
def multiply(a, b, c):
    return a * b * c

double = partial(multiply, 2)  # a=2
print(double(3, 4))  # 24

# Практический пример: сортировка
from operator import itemgetter
users = [{'name': 'Alice', 'age': 30}, {'name': 'Bob', 'age': 25}]
sorted_by_age = sorted(users, key=itemgetter('age'))
```

## wraps — Сохранение метаданных

```python
from functools import wraps

# Проблема: декоратор скрывает оригинальную функцию
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)
        print("After")
        return result
    return wrapper

@my_decorator
def greet(name):
    """Greet someone"""
    print(f"Hello, {name}!")

print(greet.__name__)  # 'wrapper'  ❌
print(greet.__doc__)   # None       ❌

# Решение: @wraps
def my_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)
        print("After")
        return result
    return wrapper

@my_decorator
def greet(name):
    """Greet someone"""
    print(f"Hello, {name}!")

print(greet.__name__)  # 'greet'   ✅
print(greet.__doc__)   # 'Greet someone'  ✅
```

## total_ordering — Генерация методов сравнения

```python
from functools import total_ordering

@total_ordering
class Version:
    def __init__(self, major, minor, patch):
        self.major = major
        self.minor = minor
        self.patch = patch

    def __eq__(self, other):
        return (self.major, self.minor, self.patch) == \
               (other.major, other.minor, other.patch)

    def __lt__(self, other):
        return (self.major, self.minor, self.patch) < \
               (other.major, other.minor, other.patch)

v1 = Version(1, 2, 0)
v2 = Version(1, 3, 0)
print(v1 < v2)   # True
print(v1 <= v2)  # True (автогенерация)
print(v1 > v2)   # False (автогенерация)
print(v1 >= v2)  # False (автогенерация)
```

> 💡 Требуется реализовать `__eq__` и один из: `__lt__`, `__le__`, `__gt__`, `__ge__`.

## singledispatch — Перегрузка по типу

```python
from functools import singledispatch

@singledispatch
def process(data):
    raise NotImplementedError("Unsupported type")

@process.register
def _(data: str):
    print(f"Processing string: {data[:50]}")

@process.register
def _(data: list):
    print(f"Processing list with {len(data)} items")

@process.register(int)
def _(data):
    print(f"Processing number: {data}")

process("hello")     # Processing string: hello
process([1, 2, 3])   # Processing list with 3 items
process(42)          # Processing number: 42
```

## Best Practices

✅ **Используйте** `@lru_cache` для мемоизации дорогих вычислений
✅ **Используйте** `@wraps` при написании декораторов
✅ **Используйте** `partial` для каррирования и фиксации аргументов
✅ **Всегда указывайте** `maxsize` в `lru_cache` для продакшена

❌ **Не используйте** `@lru_cache` для функций с побочными эффектами
❌ **Не используйте** `@lru_cache(maxsize=None)` без контроля памяти
❌ **Не забывайте** `@wraps` в декораторах — сломает docstring и имя

## Ссылки

- [Официальная документация functools](https://docs.python.org/3/library/functools.html)
- [cachetools — расширенное кэширование](https://cachetools.readthedocs.io/)

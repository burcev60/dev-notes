# Основы типизации в Python

## Описание

Python — язык с динамической типизацией, но поддерживает опциональные аннотации типов для улучшения читаемости и статического анализа.

## Аннотации переменных

```python
# Базовые аннотации
name: str = "Alice"
age: int = 30
height: float = 1.75
is_active: bool = True

# Аннотация без значения
user_id: int
user_id = 123  # OK

# Проверка mypy
# reveal_type(name)  # str
```

## Аннотации функций

```python
# Базовая сигнатура
def greet(name: str) -> str:
    return f"Hello, {name}!"

# Несколько параметров
def add(a: int, b: int) -> int:
    return a + b

# Без возвращаемого значения
def log(message: str) -> None:
    print(message)

# Значения по умолчанию
def create_user(name: str, age: int = 18, active: bool = True) -> dict:
    return {"name": name, "age": age, "active": active}
```

## Коллекции

```python
from typing import List, Dict, Set, Tuple

# List — список с типом элементов
def process(items: List[int]) -> None:
    for item in items:
        print(item)

# Dict — словарь с типами ключа и значения
def get_counts() -> Dict[str, int]:
    return {"a": 1, "b": 2}

# Set — множество
def unique(items: List[int]) -> Set[int]:
    return set(items)

# Tuple — кортеж фиксированной длины
def get_point() -> Tuple[float, float]:
    return (1.0, 2.0)

# Tuple переменной длины
def get_coords() -> Tuple[int, ...]:
    return (1, 2, 3, 4)
```

## Python 3.9+ — Встроенные дженерики

```python
# В Python 3.9+ не нужен импорт из typing
def process(items: list[int]) -> None:      # вместо List[int]
def get_data() -> dict[str, int]:            # вместо Dict[str, int]
def get_point() -> tuple[float, float]:      # вместо Tuple[float, float]
def unique(items: list[int]) -> set[int]:    # вместо Set[int]
```

## Optional и Union

```python
from typing import Optional, Union

# Optional[X] — это X или None
def find_user(user_id: int) -> Optional[str]:
    if user_id > 0:
        return "Alice"
    return None

# Union[X, Y] — один из нескольких типов
def process(value: Union[str, int]) -> str:
    return str(value)

# Python 3.10+ синтаксис
def process_new(value: str | int) -> str:
    return str(value)

def find_user_new(user_id: int) -> str | None:
    return "Alice" if user_id > 0 else None
```

## Проверка типов — mypy

```bash
# Установка
pip install mypy

# Запуск
mypy script.py
mypy src/ --ignore-missing-imports

# Строгий режим
mypy src/ --strict

# Конфигурация в pyproject.toml
[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
```

```python
# Примеры ошибок mypy

def add(a: int, b: int) -> int:
    return a + b

add(1, "2")  # error: Argument 2 has incompatible type "str"; expected "int"

def get_name() -> str:
    return 42  # error: Incompatible return value type (got "int", expected "str")
```

## Best Practices

✅ **Аннотируйте** все публичные API функции и методы
✅ **Используйте** `mypy` в CI/CD пайплайне
✅ **Используйте** `None` вместо `Optional` когда возможно
✅ **Используйте** встроенные дженерики (Python 3.9+)

❌ **Не аннотируйте** локальные переменные без необходимости
❌ **Не используйте** `Any` без веской причины
❌ **Не игнорируйте** ошибки mypy в продакшен коде

## Ссылки

- [Официальная документация typing](https://docs.python.org/3/library/typing.html)
- [mypy — статическая проверка типов](https://mypy.readthedocs.io/)
- [PEP 484 — Type Hints](https://peps.python.org/pep-0484/)
- [PEP 604 — Union types](https://peps.python.org/pep-0604/)

# typing — Аннотации типов

## Описание

Модуль `typing` предоставляет поддержку для аннотаций типов — систему статической типизации в Python.

## Базовые типы

```python
from typing import List, Dict, Set, Tuple, Optional, Union

# Коллекции
def process_items(items: List[str]) -> None:
    for item in items:
        print(item)

def get_counts() -> Dict[str, int]:
    return {'a': 1, 'b': 2}

def unique(items: List[int]) -> Set[int]:
    return set(items)

def get_point() -> Tuple[float, float]:
    return (1.0, 2.0)

# Optional — может быть None
def find_user(user_id: int) -> Optional[str]:
    if user_id > 0:
        return "Alice"
    return None

# Union — один из нескольких типов
def process(value: Union[str, int]) -> str:
    return str(value)

# Python 3.10+ синтаксис
def process_new(value: str | int) -> str:
    return str(value)
```

## Python 3.9+ — Встроенные дженерики

```python
# В Python 3.9+ можно использовать встроенные типы
def process(items: list[str]) -> None:  # вместо List[str]
    pass

def get_data() -> dict[str, int]:  # вместо Dict[str, int]
    return {}

def get_point() -> tuple[float, float]:  # вместо Tuple[float, float]
    return (1.0, 2.0)
```

## Callable — Функции как типы

```python
from typing import Callable

def apply(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)

def add(x: int, y: int) -> int:
    return x + y

result = apply(add, 5, 3)  # 8

# Callback без аргументов
def notify(callback: Callable[[], None]) -> None:
    callback()

# Lambda
apply(lambda x, y: x * y, 4, 5)  # 20
```

## Any и NoReturn

```python
from typing import Any, NoReturn

# Any — любой тип (избегайте!)
def process(value: Any) -> Any:
    return value

# NoReturn — функция не возвращает управления
def raise_error(message: str) -> NoReturn:
    raise ValueError(message)

def exit_program() -> NoReturn:
    import sys
    sys.exit(0)
```

## Generic — Обобщённые типы

```python
from typing import Generic, TypeVar

T = TypeVar('T')

class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: list[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        return self._items.pop()

# Использование
int_stack: Stack[int] = Stack()
int_stack.push(42)
# int_stack.push("text")  # mypy error!
```

## TypeVar с ограничениями

```python
from typing import TypeVar

# Ограниченный TypeVar
T = TypeVar('T', int, float)  # Только int или float

def add(a: T, b: T) -> T:
    return a + b

# TypeVar с bound
from typing import Protocol

class Comparable(Protocol):
    def __lt__(self, other: Any) -> bool: ...

T = TypeVar('T', bound=Comparable)

def minimum(a: T, b: T) -> T:
    return a if a < b else b
```

## TypedDict — Типизированные словари

```python
from typing import TypedDict

class UserDict(TypedDict):
    name: str
    age: int
    email: str

user: UserDict = {
    'name': 'Alice',
    'age': 30,
    'email': 'alice@example.com'
}

# Опциональные поля (Python 3.11+)
from typing import NotRequired

class ConfigDict(TypedDict, total=False):
    host: str
    port: int
    debug: NotRequired[bool]  # Необязательное поле
```

## TypeAlias — Псевдонимы типов

```python
from typing import TypeAlias

# Python 3.10+
UserId: TypeAlias = int
UserData: TypeAlias = dict[str, str]

def get_user(user_id: UserId) -> UserData:
    return {'name': 'Alice'}

# Сложные типы
JsonDict: TypeAlias = dict[str, Union[str, int, float, bool, None, list['JsonDict']]]
```

## Final и Literal

```python
from typing import Final, Literal

# Final — нельзя переопределить
MAX_RETRIES: Final = 3
# MAX_RETRIES = 5  # mypy error!

class Base:
    name: Final = 'base'

class Child(Base):
    # name = 'child'  # mypy error!
    pass

# Literal — конкретные значения
def set_mode(mode: Literal['read', 'write', 'append']) -> None:
    print(f"Mode: {mode}")

set_mode('read')    # OK
# set_mode('delete')  # mypy error!
```

## cast — Приведение типов

```python
from typing import cast

# Когда type checker не может определить тип
def get_data() -> object:
    return [1, 2, 3]

data = get_data()
# data.append(4)  # mypy error: object не имеет append

items = cast(list[int], data)
items.append(4)  # OK
```

> ⚠️ `cast()` — это только подсказка для type checker, runtime не проверяет!

## Best Practices

✅ **Используйте** аннотации во всех публичных API
✅ **Используйте** `mypy` или `pyright` для проверки
✅ **Используйте** `TypedDict` вместо `dict[str, Any]`
✅ **Используйте** `Literal` для ограниченных наборов значений

❌ **Избегайте** `Any` — это отключение проверки типов
❌ **Не используйте** `cast()` без необходимости
❌ **Не игнорируйте** ошибки mypy в продакшен коде

## Ссылки

- [Официальная документация typing](https://docs.python.org/3/library/typing.html)
- [mypy — статическая проверка типов](https://mypy.readthedocs.io/)
- [PEP 484 — Type Hints](https://peps.python.org/pep-0484/)

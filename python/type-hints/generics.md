# Generics — Обобщённые типы

## Описание

Дженерики (generics) позволяют создавать переиспользуемые компоненты с параметризованными типами.

## TypeVar — Параметр типа

```python
from typing import TypeVar

# Простой TypeVar
T = TypeVar('T')

def first(items: list[T]) -> T:
    return items[0]

# Использование
names: list[str] = ['Alice', 'Bob']
first_name: str = first(names)  # T выводится как str

numbers: list[int] = [1, 2, 3]
first_num: int = first(numbers)  # T выводится как int
```

## TypeVar с ограничениями

```python
from typing import TypeVar

# Только определённые типы
T = TypeVar('T', int, float)

def add(a: T, b: T) -> T:
    return a + b

add(1, 2)        # OK, T = int
add(1.5, 2.5)    # OK, T = float
# add(1, 2.5)    # Ошибка: разные типы

# Bound — ограничение сверху
from typing import Protocol

class Comparable(Protocol):
    def __lt__(self, other: object) -> bool: ...

T = TypeVar('T', bound=Comparable)

def minimum(a: T, b: T) -> T:
    return a if a < b else b
```

## Generic классы

```python
from typing import Generic, TypeVar

T = TypeVar('T')

class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: list[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        if not self._items:
            raise IndexError("Pop from empty stack")
        return self._items.pop()

    def peek(self) -> T:
        return self._items[-1]

# Использование
int_stack: Stack[int] = Stack()
int_stack.push(42)
# int_stack.push("text")  # mypy error!

str_stack: Stack[str] = Stack()
str_stack.push("hello")
```

## Generic с несколькими параметрами

```python
from typing import Generic, TypeVar

K = TypeVar('K')
V = TypeVar('V')

class KeyValueStore(Generic[K, V]):
    def __init__(self) -> None:
        self._data: dict[K, V] = {}

    def set(self, key: K, value: V) -> None:
        self._data[key] = value

    def get(self, key: K) -> V | None:
        return self._data.get(key)

# Использование
store: KeyValueStore[str, int] = KeyValueStore()
store.set("counter", 42)
count: int | None = store.get("counter")
```

## Наследование Generic

```python
from typing import Generic, TypeVar

T = TypeVar('T')

class Repository(Generic[T]):
    def get(self, id: int) -> T:
        raise NotImplementedError

class User:
    def __init__(self, id: int, name: str):
        self.id = id
        self.name = str

class UserRepository(Repository[User]):
    def __init__(self) -> None:
        self._users: dict[int, User] = {}

    def get(self, id: int) -> User:
        return self._users[id]
```

## Covariant и Contravariant

```python
from typing import Generic, TypeVar, Sequence

# Covariant — для чтения (output)
T_co = TypeVar('T_co', covariant=True)

class Producer(Generic[T_co]):
    def produce(self) -> T_co:
        ...

# Contravariant — для записи (input)
T_contra = TypeVar('T_contra', contravariant=True)

class Consumer(Generic[T_contra]):
    def consume(self, item: T_contra) -> None:
        ...

# Практический пример
class Animal: pass
class Dog(Animal): pass

# Producer[Dog] — подтип Producer[Animal] (covariant)
# Consumer[Animal] — подтип Consumer[Dog] (contravariant)
```

## Generic функции

```python
from typing import TypeVar, Callable

T = TypeVar('T')
R = TypeVar('R')

def compose(f: Callable[[T], R], g: Callable[[R], T]) -> Callable[[T], T]:
    return lambda x: f(g(x))

# Использование
def to_str(x: int) -> str:
    return str(x)

def parse_int(s: str) -> int:
    return int(s)

identity = compose(parse_int, to_str)
```

## Best Practices

✅ **Используйте** `T`, `K`, `V` как имена TypeVar по умолчанию
✅ **Используйте** `Generic[T]` для классов-контейнеров
✅ **Используйте** `covariant=True` для read-only контейнеров
✅ **Аннотируйте** все методы в Generic классах

❌ **Не используйте** `Any` внутри Generic без необходимости
❌ **Не смешивайте** covariant и contravariant неправильно
❌ **Не забывайте** что TypeVar должен быть объявлен на уровне модуля

## Ссылки

- [Официальная документация Generic](https://docs.python.org/3/library/typing.html#typing.Generic)
- [PEP 484 — Generics](https://peps.python.org/pep-0484/#generics)
- [PEP 695 — Type Parameter Syntax (Python 3.12+)](https://peps.python.org/pep-0695/)

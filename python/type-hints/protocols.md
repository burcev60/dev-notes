# Protocols — Структурная типизация

## Описание

Protocols позволяют использовать структурную типизацию (duck typing) в статической проверке типов — "если это ходит как утка и крякает как утка, это утка".

## Базовое использование

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None:
        ...

class Circle:
    def draw(self) -> None:
        print("Drawing circle")

class Square:
    def draw(self) -> None:
        print("Drawing square")

# Функция принимает любой объект с методом draw()
def render(shape: Drawable) -> None:
    shape.draw()

render(Circle())   # OK
render(Square())   # OK

# Класс без метода draw()
class Line:
    pass

# render(Line())  # mypy error!
```

## Protocol с атрибутами

```python
from typing import Protocol

class UserLike(Protocol):
    name: str
    age: int
    is_active: bool

def activate(user: UserLike) -> None:
    if not user.is_active:
        print(f"Activating {user.name}")
        user.is_active = True

# Dataclass подходит автоматически
from dataclasses import dataclass

@dataclass
class User:
    name: str
    age: int
    is_active: bool

user = User("Alice", 30, False)
activate(user)  # OK
```

## Protocol с методами

```python
from typing import Protocol, Iterator

class Iterable(Protocol):
    def __iter__(self) -> Iterator:
        ...

class Sized(Protocol):
    def __len__(self) -> int:
        ...

class SizedIterable(Iterable, Sized, Protocol):
    """Объединение нескольких протоколов"""
    ...

def process(items: SizedIterable) -> None:
    print(f"Processing {len(items)} items")
    for item in items:
        print(item)

# Встроенные типы подходят
process([1, 2, 3])           # OK
process({1, 2, 3})           # OK
process("hello")             # OK
# process(42)                # mypy error!
```

## Generic Protocol

```python
from typing import Protocol, TypeVar, Generic

T = TypeVar('T')

class Repository(Protocol[T]):
    def get(self, id: int) -> T:
        ...
    def save(self, item: T) -> None:
        ...

class User:
    def __init__(self, id: int, name: str):
        self.id = id
        self.name = name

class UserRepository:
    def __init__(self) -> None:
        self._users: dict[int, User] = {}

    def get(self, id: int) -> User:
        return self._users[id]

    def save(self, user: User) -> None:
        self._users[user.id] = user

def process_user(repo: Repository[User]) -> None:
    user = repo.get(1)
    print(user.name)

process_user(UserRepository())  # OK
```

## Runtime checkable

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Closable(Protocol):
    def close(self) -> None:
        ...

class File:
    def close(self) -> None:
        pass

file = File()

# isinstance проверка (только для @runtime_checkable)
if isinstance(file, Closable):
    file.close()

# ⚠️ isinstance проверяет только наличие атрибутов, не сигнатуры!
```

## Стандартные протоколы

```python
from typing import Iterable, Iterator, Sequence, Mapping

# Iterable — можно итерировать
def process(items: Iterable[int]) -> None:
    for item in items:
        print(item)

# Iterator — итератор
def get_next(it: Iterator[int]) -> int:
    return next(it)

# Sequence — индексированная коллекция
def get_item(seq: Sequence[str], idx: int) -> str:
    return seq[idx]

# Mapping — словарь
def get_value(mapping: Mapping[str, int], key: str) -> int:
    return mapping[key]
```

## Protocol vs ABC

```python
# ABC — Nominal typing (явное наследование)
from abc import ABC, abstractmethod

class AnimalABC(ABC):
    @abstractmethod
    def speak(self) -> None:
        ...

class Dog(AnimalABC):
    def speak(self) -> None:
        print("Woof!")

# Dog должен явно наследовать AnimalABC
# dog: AnimalABC = Dog()  # OK

# Protocol — Structural typing (совпадение интерфейса)
from typing import Protocol

class Speakable(Protocol):
    def speak(self) -> None:
        ...

class Cat:
    def speak(self) -> None:
        print("Meow!")

# Cat автоматически подходит под Speakable
# cat: Speakable = Cat()  # OK, без наследования!
```

## Best Practices

✅ **Используйте** Protocols для loose coupling между компонентами
✅ **Используйте** `@runtime_checkable` только когда нужна runtime проверка
✅ **Используйте** Protocols для тестирования моков
✅ **Комбинируйте** несколько Protocol через наследование

❌ **Не используйте** `@runtime_checkable` для производительности (isinstance медленный)
❌ **Не добавляйте** реализации в Protocol (только сигнатуры)
❌ **Не путайте** Protocol с ABC — разные подходы

## Ссылки

- [Официальная документация Protocol](https://docs.python.org/3/library/typing.html#typing.Protocol)
- [PEP 544 — Protocols](https://peps.python.org/pep-0544/)
- [typing_extensions — Protocol для старых Python](https://pypi.org/project/typing-extensions/)

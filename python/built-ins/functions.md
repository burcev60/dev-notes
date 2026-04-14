# Встроенные функции Python

## Описание

Python предоставляет набор встроенных функций, доступных без импорта модулей.

## Итерация

```python
# enumerate — индекс и значение
fruits = ['apple', 'banana', 'cherry']
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")
# 0: apple, 1: banana, 2: cherry

# start — сдвиг индекса
for i, fruit in enumerate(fruits, start=1):
    print(f"{i}: {fruit}")

# zip — параллельная итерация
names = ['Alice', 'Bob', 'Charlie']
ages = [30, 25, 35]
for name, age in zip(names, ages):
    print(f"{name}: {age}")

# strict=True (Python 3.10+) — ошибка если разная длина
# zip(names, ages, strict=True)

# reversed — обратный порядок
for item in reversed([1, 2, 3]):
    print(item)  # 3, 2, 1

# iter — создание итератора
it = iter([1, 2, 3])
next(it)  # 1
next(it)  # 2
```

## map, filter, reduce

```python
# map — применение функции к каждому элементу
numbers = [1, 2, 3, 4]
squares = list(map(lambda x: x**2, numbers))
# [1, 4, 9, 16]

# Несколько итерируемых
list(map(pow, [2, 3], [3, 2]))  # [8, 9]

# filter — фильтрация
evens = list(filter(lambda x: x % 2 == 0, numbers))
# [2, 4]

# reduce — свёртка (из functools)
from functools import reduce
product = reduce(lambda x, y: x * y, numbers)
# 24 (1*2*3*4)

# С начальным значением
total = reduce(lambda x, y: x + y, numbers, 100)
# 110
```

## sorted, min, max

```python
# sorted — сортировка (возвращает новый список)
numbers = [3, 1, 4, 1, 5]
sorted_nums = sorted(numbers)  # [1, 1, 3, 4, 5]
sorted_desc = sorted(numbers, reverse=True)  # [5, 4, 3, 1, 1]

# key — функция для сравнения
users = [{'name': 'Alice', 'age': 30}, {'name': 'Bob', 'age': 25}]
sorted_by_age = sorted(users, key=lambda u: u['age'])

from operator import itemgetter
sorted_by_age = sorted(users, key=itemgetter('age'))

# min/max
min_val = min([3, 1, 4, 1, 5])  # 1
max_val = max([3, 1, 4, 1, 5])  # 5

# С key
youngest = min(users, key=lambda u: u['age'])
oldest = max(users, key=lambda u: u['age'])

# default — значение по умолчанию (Python 3.4+)
min([], default=0)  # 0
```

## any, all

```python
# any — True если хоть один True
any([False, False, True])   # True
any([])                      # False

# all — True если все True
all([True, True, True])     # True
all([True, False, True])    # False
all([])                      # True (vacuous truth)

# Практические примеры
users = [{'active': True}, {'active': False}]
any_active = any(u['active'] for u in users)  # True
all_active = all(u['active'] for u in users)  # False

# Валидация
numbers = [1, 2, 3, 4]
all_positive = all(n > 0 for n in numbers)  # True
has_even = any(n % 2 == 0 for n in numbers)  # True
```

## sum, len, abs

```python
# sum — сумма
sum([1, 2, 3, 4])           # 10
sum([1, 2, 3, 4], 100)      # 110 (с начальным значением)

# len — длина
len([1, 2, 3])              # 3
len("hello")                # 5
len({'a': 1, 'b': 2})       # 2

# abs — абсолютное значение
abs(-42)                    # 42
abs(3.14)                   # 3.14
abs(3+4j)                   # 5.0 (модуль комплексного)
```

## round, pow, divmod

```python
# round — округление
round(3.14159, 2)           # 3.14
round(2.675, 2)             # 2.67 (floating point issue!)

# pow — возведение в степень
pow(2, 3)                   # 8 (эквивалент 2**3)
pow(2, 3, 5)                # 3 (эквивалент (2**3) % 5)

# divmod — частное и остаток
divmod(17, 5)               # (3, 2)
quotient, remainder = divmod(17, 5)
```

## isinstance, issubclass, type

```python
# isinstance — проверка типа
isinstance(42, int)         # True
isinstance("hello", str)    # True
isinstance([1, 2], (list, tuple))  # True (один из)

# issubclass — проверка наследования
class Animal: pass
class Dog(Animal): pass

issubclass(Dog, Animal)     # True
issubclass(Animal, Dog)     # False

# type — тип объекта
type(42)                    # <class 'int'>
type("hello")               # <class 'str'>

# Сравнение типов
type(42) is int             # True
```

## hasattr, getattr, setattr, delattr

```python
class User:
    def __init__(self):
        self.name = "Alice"
        self.age = 30

user = User()

# hasattr — проверка атрибута
hasattr(user, 'name')       # True
hasattr(user, 'email')      # False

# getattr — получение с дефолтом
getattr(user, 'name')       # 'Alice'
getattr(user, 'email', 'N/A')  # 'N/A' (дефолт)

# setattr — установка
setattr(user, 'email', 'alice@example.com')
user.email                  # 'alice@example.com'

# delattr — удаление
delattr(user, 'age')
hasattr(user, 'age')        # False
```

## vars, dir, help

```python
# vars — dict атрибутов
user = {'name': 'Alice'}
vars(user)                  # {'name': 'Alice'}

# dir — список атрибутов
dir([1, 2, 3])              # ['__add__', 'append', ...]

# help — документация
help(str.upper)             # Покажет docstring
```

## Best Practices

✅ **Используйте** `enumerate()` вместо `range(len(list))`
✅ **Используйте** `zip()` для параллельной итерации
✅ **Используйте** list comprehension вместо `map()`/`filter()` для читаемости
✅ **Используйте** `any()`/`all()` с генераторами (ленивые)

❌ **Не используйте** `type(x) == int` — используйте `isinstance(x, int)`
❌ **Не используйте** `hasattr()` перед `getattr()` — используйте `getattr(obj, attr, default)`

## Ссылки

- [Официальная документация built-in functions](https://docs.python.org/3/library/functions.html)

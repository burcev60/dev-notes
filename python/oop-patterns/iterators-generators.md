# Итераторы и генераторы

## Описание

Итераторы и генераторы — инструменты для эффективной работы с последовательностями данных без загрузки всего объёма в память.

## Итераторы

```python
# Итератор — объект с __iter__ и __next__
class Counter:
    def __init__(self, max: int) -> None:
        self.max = max
        self.current = 0

    def __iter__(self):
        return self

    def __next__(self) -> int:
        if self.current >= self.max:
            raise StopIteration
        value = self.current
        self.current += 1
        return value

# Использование
counter = Counter(5)
for num in counter:
    print(num)  # 0, 1, 2, 3, 4

# Встроенные функции
next(counter)  # Ручной вызов
iter([1, 2, 3])  # Создание итератора из списка
```

## Генераторы

```python
# Генератор-функция (yield)
def count_up(max: int):
    current = 0
    while current < max:
        yield current
        current += 1

# Использование
for num in count_up(5):
    print(num)  # 0, 1, 2, 3, 4

# Генератор возвращает generator object
gen = count_up(3)
print(type(gen))  # <class 'generator'>
print(next(gen))  # 0
print(next(gen))  # 1
print(next(gen))  # 2
# print(next(gen))  # StopIteration
```

## yield from — Делегирование

```python
def chain(*iterables):
    """Объединение нескольких итераторов"""
    for iterable in iterables:
        yield from iterable

result = list(chain([1, 2], [3, 4], [5]))
# [1, 2, 3, 4, 5]

# Рекурсивный генератор
def flatten(nested):
    """Flatten вложенного списка"""
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)
        else:
            yield item

nested = [1, [2, 3], [4, [5, 6]]]
print(list(flatten(nested)))  # [1, 2, 3, 4, 5, 6]
```

## Генераторные выражения

```python
# Синтаксис (круглые скобки)
squares = (x**2 for x in range(10))
print(next(squares))  # 0
print(next(squares))  # 1

# В функциях (скобки не нужны)
total = sum(x**2 for x in range(10))

# Фильтрация
evens = (x for x in range(20) if x % 2 == 0)

# Несколько условий
result = (x for x in range(100) if x % 2 == 0 if x % 3 == 0)

# С несколькими iterables
pairs = ((x, y) for x in range(3) for y in range(3))
print(list(pairs))  # [(0,0), (0,1), (0,2), (1,0), ...]
```

## Отправка значений в генератор

```python
def accumulator():
    """Генератор, принимающий значения"""
    total = 0
    while True:
        value = yield total
        if value is not None:
            total += value

gen = accumulator()
print(next(gen))       # 0 (инициализация)
print(gen.send(10))    # 10
print(gen.send(20))    # 30
print(gen.send(5))     # 35

# Закрытие
gen.close()
```

## Практические примеры

```python
import os
from pathlib import Path

# Чтение больших файлов построчно
def read_large_file(filename: str):
    """Чтение файла без загрузки в память"""
    with open(filename, 'r') as f:
        for line in f:
            yield line.strip()

# Поиск файлов
def find_files(directory: str, pattern: str):
    """Рекурсивный поиск файлов по паттерну"""
    for root, _, files in os.walk(directory):
        for file in files:
            if pattern in file:
                yield os.path.join(root, file)

# Пагинация
def paginate(items, page_size: int):
    """Разбиение на страницы"""
    for i in range(0, len(items), page_size):
        yield items[i:i + page_size]

for page in paginate(range(100), 10):
    print(page)  # [0..9], [10..19], ...
```

## itertools + генераторы

```python
from itertools import islice, tee

# Первые N элементов генератора
def take(n, iterable):
    return list(islice(iterable, n))

# Peek — посмотреть не потребляя
def peek(iterator, n=1):
    it, peeker = tee(iterator)
    return take(n, peeker), it

gen = (x**2 for x in range(100))
first_5, gen = peek(gen, 5)
print(first_5)  # [0, 1, 4, 9, 16]
print(take(5, gen))  # [25, 36, 49, 64, 81]
```

## Best Practices

✅ **Используйте** генераторы для больших данных (экономия памяти)
✅ **Используйте** `yield from` для делегирования
✅ **Используйте** генераторные выражения вместо list comprehension когда возможно
✅ **Закрывайте** генераторы через `close()` или `with`

❌ **Не пытайтесь** переиспользовать исчерпанный генератор
❌ **Не загружайте** всё в генератор если нужен random access
❌ **Не используйте** `list(generator)` если данных очень много

## Ссылки

- [Официальная документация iterators](https://docs.python.org/3/library/stdtypes.html#iterator-types)
- [Официальная документация generators](https://docs.python.org/3/tutorial/classes.html#generators)
- [PEP 255 — Simple Generators](https://peps.python.org/pep-0255/)
- [PEP 380 — yield from](https://peps.python.org/pep-0380/)

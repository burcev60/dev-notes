# itertools — Итераторы и комбинации

## Описание

Модуль `itertools` предоставляет эффективные инструменты для работы с итераторами — комбинации, группировки, бесконечные последовательности.

## Бесконечные итераторы

```python
from itertools import count, cycle, repeat

# count — счётчик
for i in count(10):
    if i > 15:
        break
    print(i)  # 10, 11, 12, 13, 14, 15

# count с шагом
list(zip(range(3), count(100, 10)))
# [(0, 100), (1, 110), (2, 120)]

# cycle — бесконечное повторение
colors = cycle(['red', 'green', 'blue'])
[next(colors) for _ in range(5)]
# ['red', 'green', 'blue', 'red', 'green']

# repeat — повторение значения
list(repeat('A', 3))  # ['A', 'A', 'A']
```

## Комбинации и перестановки

```python
from itertools import combinations, permutations, product

# combinations — сочетания (порядок не важен)
list(combinations('ABC', 2))
# [('A', 'B'), ('A', 'C'), ('B', 'C')]

# combinations_with_replacement — с повторами
list(combinations_with_replacement('AB', 2))
# [('A', 'A'), ('A', 'B'), ('B', 'B')]

# permutations — перестановки (порядок важен)
list(permutations('AB', 2))
# [('A', 'B'), ('B', 'A')]

# product — декартово произведение
list(product('AB', '12'))
# [('A', '1'), ('A', '2'), ('B', '1'), ('B', '2')]

# product с repeat
list(product('AB', repeat=2))
# [('A', 'A'), ('A', 'B'), ('B', 'A'), ('B', 'B')]
```

## groupby — Группировка

```python
from itertools import groupby

# Группировка (данные должны быть отсортированы!)
data = [
    ('fruit', 'apple'),
    ('fruit', 'banana'),
    ('veg', 'carrot'),
]

# Сортировка по ключу
data.sort(key=lambda x: x[0])

for key, group in groupby(data, key=lambda x: x[0]):
    print(f"{key}: {list(group)}")
# fruit: [('fruit', 'apple'), ('fruit', 'banana')]
# veg: [('veg', 'carrot')]

# Группировка чисел
numbers = [1, 2, 2, 3, 3, 3, 1]
for key, group in groupby(numbers):
    print(f"{key}: {len(list(group))}")
# 1: 1, 2: 2, 3: 3, 1: 1
```

## chain — Цепочка итераторов

```python
from itertools import chain

# Объединение итераторов
list(chain([1, 2], [3, 4], [5]))
# [1, 2, 3, 4, 5]

# chain.from_iterable — из одного итерируемого
list(chain.from_iterable([[1, 2], [3, 4]]))
# [1, 2, 3, 4]

# Практический пример: flattening
nested = [[1, 2], [3, 4], [5]]
flat = list(chain.from_iterable(nested))
# [1, 2, 3, 4, 5]
```

## islice, tee, zip_longest

```python
from itertools import islice, tee, zip_longest

# islice — срез итератора
list(islice(range(10), 2, 6))    # [2, 3, 4, 5]
list(islice(range(10), None, 5)) # [0, 1, 2, 3, 4]

# tee — клонирование итератора
it1, it2 = tee(range(5))
list(it1)  # [0, 1, 2, 3, 4]
list(it2)  # [0, 1, 2, 3, 4]

# zip_longest — zip с заполнением
list(zip_longest('AB', '123', fillvalue='-'))
# [('A', '1'), ('B', '2'), ('-', '3')]
```

## takewhile, dropwhile, compress

```python
from itertools import takewhile, dropwhile, compress

# takewhile — пока условие истинно
list(takewhile(lambda x: x < 5, range(10)))
# [0, 1, 2, 3, 4]

# dropwhile — пропуска пока условие истинно
list(dropwhile(lambda x: x < 5, range(10)))
# [5, 6, 7, 8, 9]

# compress — фильтрация по маске
data = ['A', 'B', 'C', 'D']
selectors = [1, 0, 1, 0]
list(compress(data, selectors))  # ['A', 'C']
```

## Best Practices

✅ **Используйте** `chain.from_iterable()` для flattening вложенных списков
✅ **Используйте** `groupby()` для группировки (не забудьте отсортировать!)
✅ **Используйте** `combinations()` для генерации пар/троек
✅ **Используйте** `islice()` для эффективных срезов больших данных

❌ **Не забывайте** сортировать перед `groupby()`
❌ **Не используйте** `tee()` если итератор уже частично потреблён

## Ссылки

- [Официальная документация itertools](https://docs.python.org/3/library/itertools.html)
- [More Itertools — дополнительные инструменты](https://more-itertools.readthedocs.io/)

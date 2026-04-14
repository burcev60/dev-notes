# collections — Контейнеры данных

## Описание

Модуль `collections` предоставляет специализированные типы контейнеров — альтернативы встроенным `dict`, `list`, `tuple`.

## namedtuple — Именованные кортежи

```python
from collections import namedtuple

# Определение
Point = namedtuple('Point', ['x', 'y'])
# или через строку
Point = namedtuple('Point', 'x y')

# Использование
p = Point(10, 20)
print(p.x, p.y)       # 10 20
print(p[0], p[1])     # 10 20 (обратная совместимость)

# Со значениями по умолчанию
User = namedtuple('User', ['name', 'age', 'role'], defaults=['user'])
u = User('Alice', 30)
print(u.role)  # 'user'

# Преобразование в dict
print(p._asdict())  # {'x': 10, 'y': 20}
```

> 💡 В Python 3.7+ обычные `dataclasses` часто предпочтительнее.

## defaultdict — Словарь со значением по умолчанию

```python
from collections import defaultdict

# Автоматическое создание значений
d = defaultdict(list)
d['users'].append('Alice')
d['users'].append('Bob')
# {'users': ['Alice', 'Bob']}

# Счётчик
counts = defaultdict(int)
for word in ['apple', 'banana', 'apple']:
    counts[word] += 1
# {'apple': 2, 'banana': 1}

# Вложенные структуры
tree = lambda: defaultdict(tree)
users = tree()
users['Alice']['email'] = 'alice@example.com'

# Фабрика значений
d = defaultdict(lambda: 'N/A')
print(d['missing'])  # 'N/A'
```

## Counter — Счётчик

```python
from collections import Counter

# Подсчёт
c = Counter(['a', 'b', 'a', 'c', 'a', 'b'])
print(c)           # Counter({'a': 3, 'b': 2, 'c': 1})

# Из строки
c = Counter("hello world")
print(c.most_common(3))  # [('l', 3), ('o', 2), ('h', 1)]

# Операции
c1 = Counter(a=3, b=1)
c2 = Counter(a=1, b=2)
print(c1 + c2)     # Counter({'a': 4, 'b': 3})
print(c1 - c2)     # Counter({'a': 2})

# Элементы
c = Counter(a=2, b=1)
list(c.elements())  # ['a', 'a', 'b']
```

## deque — Двусторонняя очередь

```python
from collections import deque

# Создание
dq = deque([1, 2, 3])
dq = deque(maxlen=5)  # Ограниченная

# Операции
dq.append(4)       # В конец
dq.appendleft(0)   # В начало
dq.pop()           # Из конца → 4
dq.popleft()       # Из начала → 0

# Вращение
dq = deque([1, 2, 3, 4, 5])
dq.rotate(2)   # [4, 5, 1, 2, 3]
dq.rotate(-1)  # [5, 1, 2, 3, 4]

# Ограниченная очередь (автоматическое вытеснение)
dq = deque(maxlen=3)
dq.append(1)
dq.append(2)
dq.append(3)
dq.append(4)  # 1 удалён автоматически
# deque([2, 3, 4], maxlen=3)
```

## Best Practices

✅ **Используйте** `defaultdict(list)` для группировки
✅ **Используйте** `Counter` для подсчёта элементов
✅ **Используйте** `deque` для очередей и буферов (O(1) с обоих концов)
✅ **Используйте** `deque(maxlen)` для sliding window

❌ **Не используйте** `list` как очередь (`pop(0)` — O(n))
❌ **Не используйте** `namedtuple` если нужен изменяемый объект

## Ссылки

- [Официальная документация collections](https://docs.python.org/3/library/collections.html)
- [dataclass — современная альтернатива namedtuple](https://docs.python.org/3/library/dataclasses.html)

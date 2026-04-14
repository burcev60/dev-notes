# contextlib — Контекстные менеджеры

## Описание

Модуль `contextlib` предоставляет утилиты для работы с контекстными менеджерами и создания собственных через декораторы.

## contextmanager — Генератор как контекстный менеджер

```python
from contextlib import contextmanager

# Простой пример
@contextmanager
def managed_resource(name):
    print(f"Acquiring {name}")
    resource = {'name': name, 'status': 'open'}
    try:
        yield resource
    finally:
        print(f"Releasing {name}")
        resource['status'] = 'closed'

# Использование
with managed_resource('database') as db:
    print(f"Using {db['name']}")
    # Acquiring database
    # Using database
    # Releasing database

# Файл-подобный объект
@contextmanager
def open_file(filename, mode):
    f = open(filename, mode)
    try:
        yield f
    finally:
        f.close()

with open_file('data.txt', 'w') as f:
    f.write('Hello')
```

## Практические примеры contextmanager

```python
from contextlib import contextmanager
import time

# Замер времени
@contextmanager
def timer(name):
    start = time.time()
    try:
        yield
    finally:
        elapsed = time.time() - start
        print(f"{name} took {elapsed:.3f}s")

with timer("Processing"):
    time.sleep(1)  # Processing took 1.002s

# Temporary change directory
import os
from pathlib import Path

@contextmanager
def change_dir(path):
    old_cwd = os.getcwd()
    os.chdir(path)
    try:
        yield
    finally:
        os.chdir(old_cwd)

with change_dir('/tmp'):
    print(os.getcwd())  # /tmp

# Transaction manager
@contextmanager
def transaction(db):
    db.begin()
    try:
        yield
        db.commit()
    except Exception:
        db.rollback()
        raise
```

## suppress — Подавление исключений

```python
from contextlib import suppress
import os

# Удаление файла (игнорирование ошибки)
with suppress(FileNotFoundError):
    os.remove('nonexistent.txt')

# Эквивалент:
try:
    os.remove('nonexistent.txt')
except FileNotFoundError:
    pass

# Несколько типов
with suppress(FileNotFoundError, PermissionError):
    os.remove('file.txt')
```

## ExitStack — Динамическое управление контекстами

```python
from contextlib import ExitStack
import tempfile

# Открытие нескольких файлов
with ExitStack() as stack:
    files = [stack.enter_context(open(f'file_{i}.txt', 'w')) for i in range(3)]
    files[0].write('First')
    files[1].write('Second')
    # Все файлы закроются автоматически

# Динамическое добавление
with ExitStack() as stack:
    if condition:
        stack.enter_context(some_context())
    # Cleanup при выходе

# Callback при выходе
with ExitStack() as stack:
    temp_dir = tempfile.mkdtemp()
    stack.callback(shutil.rmtree, temp_dir)
    # temp_dir будет удалён при выходе
```

## closing — Автозакрытие

```python
from contextlib import closing
import urllib.request

# Для объектов с методом close(), но без __enter__/__exit__
with closing(urllib.request.urlopen('http://example.com')) as response:
    data = response.read()

# Эквивалент:
response = urllib.request.urlopen('http://example.com')
try:
    data = response.read()
finally:
    response.close()
```

## asynccontextmanager — Асинхронный контекстный менеджер

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def async_db_session():
    session = create_session()
    try:
        yield session
        await session.commit()
    except Exception:
        await session.rollback()
        raise
    finally:
        await session.close()

# Использование
async def get_user():
    async with async_db_session() as session:
        return await session.get(User, 1)
```

## Best Practices

✅ **Используйте** `@contextmanager` для простых менеджеров ресурсов
✅ **Используйте** `suppress()` для безопасного игнорирования ошибок
✅ **Используйте** `ExitStack` для динамического управления контекстами
✅ **Всегда** обрабатывайте исключения в `finally` блоке

❌ **Не используйте** `suppress()` для скрытия реальных ошибок
❌ **Не забывайте** `try/finally` в `@contextmanager` функциях

## Ссылки

- [Официальная документация contextlib](https://docs.python.org/3/library/contextlib.html)
- [Контекстные менеджеры в Python](https://docs.python.org/3/reference/datamodel.html#context-managers)

# os и os.path — Работа с файловой системой

## Описание

Модули `os` и `os.path` предоставляют функции для взаимодействия с операционной системой и работы с файловой системой.

## Основные функции os

### Работа с директориями

```python
import os

# Текущая рабочая директория
cwd = os.getcwd()
print(f"Current: {cwd}")

# Смена директории
os.chdir('/path/to/dir')

# Создание директории
os.mkdir('new_dir')           # Одна директория
os.makedirs('a/b/c')          # Рекурсивно

# Удаление
os.rmdir('new_dir')           # Пустую директорию
os.removedirs('a/b/c')        # Рекурсивно (пустые)

# Список файлов
files = os.listdir('/path')   # Все entries
for entry in os.scandir('/path'):
    if entry.is_file():
        print(entry.name)
```

### Работа с файлами

```python
import os

# Удаление файла
os.remove('file.txt')

# Переименование
os.rename('old.txt', 'new.txt')

# Проверка существования
if os.path.exists('file.txt'):
    print("Exists!")

# Информация о файле
stat = os.stat('file.txt')
print(f"Size: {stat.st_size} bytes")
print(f"Modified: {stat.st_mtime}")
```

### Переменные окружения

```python
import os

# Получить переменную
db_url = os.getenv('DATABASE_URL')
db_url = os.environ.get('DATABASE_URL')

# Установить
os.environ['MY_VAR'] = 'value'

# Проверить наличие
if 'MY_VAR' in os.environ:
    print("Set!")
```

### os.path — утилиты для путей

```python
import os.path

#_join
path = os.path.join('dir', 'subdir', 'file.txt')
# 'dir/subdir/file.txt'

# basename
os.path.basename('/path/to/file.txt')  # 'file.txt'

# dirname
os.path.dirname('/path/to/file.txt')   # '/path/to'

# split
os.path.splitext('file.tar.gz')        # ('file.tar', '.gz')

# absolute
os.path.abspath('../file.txt')

# is_file / is_dir
os.path.isfile('file.txt')   # True/False
os.path.isdir('/path')       # True/False
```

## Best Practices

✅ **Используйте** `os.path.join()` для построения путей (кроссплатформенность)
✅ **Используйте** `os.path.exists()` перед операциями с файлами
✅ **Используйте** `os.scandir()` вместо `os.listdir()` для производительности

❌ **Не хардкодьте** пути с `/` — используйте `os.path.join()`
❌ **Не забывайте** обрабатывать `FileNotFoundError`

## Современная альтернатива

Для нового кода рассмотрите `pathlib` — более современный ООП-подход:

```python
from pathlib import Path

path = Path('dir') / 'file.txt'
path.exists()
path.read_text()
```

## Ссылки

- [Официальная документация os](https://docs.python.org/3/library/os.html)
- [Официальная документация os.path](https://docs.python.org/3/library/os.path.html)
- [pathlib — современная альтернатива](https://docs.python.org/3/library/pathlib.html)

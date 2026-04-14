# sys — Параметры среды и аргументы

## Описание

Модуль `sys` предоставляет доступ к переменным и функциям, взаимодействующим с интерпретатором Python.

## Аргументы командной строки

```python
import sys

# Список аргументов (первый — имя скрипта)
print(sys.argv)
# python script.py arg1 arg2 → ['script.py', 'arg1', 'arg2']

# Пример разбора
if len(sys.argv) > 1:
    filename = sys.argv[1]
    print(f"Processing: {filename}")
```

> 💡 Для сложных CLI рассмотрите `argparse` или `click`.

## Стандартные потоки

```python
import sys

# stdout — стандартный вывод
sys.stdout.write("Hello\n")
print("Hello")  # эквивалент

# stderr — стандартный поток ошибок
sys.stderr.write("Error occurred\n")
print("Error", file=sys.stderr)

# stdin — стандартный ввод
for line in sys.stdin:
    print(f"Input: {line.strip()}")
# Использование: echo "data" | python script.py
```

## Переменные интерпретатора

```python
import sys

# Версия Python
print(sys.version)        # Полная строка
print(sys.version_info)   # Кортеж: (3, 11, 4, 'final', 0)

# Проверка версии
if sys.version_info >= (3, 10):
    print("Python 3.10+")

# Путь поиска модулей
print(sys.path)
# ['/home/user/project', '/usr/lib/python311.zip', ...]

# Добавление пути
sys.path.insert(0, '/custom/path')

# Платформа
print(sys.platform)  # 'linux', 'win32', 'darwin'
```

## Завершение программы

```python
import sys

# Выход с кодом
sys.exit(0)   # Успех
sys.exit(1)   # Ошибка

# Выход с сообщением
sys.exit("Something went wrong")
```

## Рекурсия

```python
import sys

# Лимит глубины рекурии
print(sys.getrecursionlimit())  # Обычно 1000

# Установка своего лимита
sys.setrecursionlimit(2000)
```

## Best Practices

✅ **Используйте** `sys.argv` для простых скриптов
✅ **Используйте** `sys.stderr` для ошибок и предупреждений
✅ **Используйте** `sys.exit()` для явного завершения

❌ **Не модифицируйте** `sys.path` без необходимости — это антипаттер

❌ **Не меняйте** `sys.setrecursionlimit()` без веской причины

## Ссылки

- [Официальная документация sys](https://docs.python.org/3/library/sys.html)
- [argparse — продвинутый CLI](https://docs.python.org/3/library/argparse.html)

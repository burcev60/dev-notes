# uv — Скрипты (PEP 723)

## Описание

PEP 723 — inline метаданные для Python скриптов. Позволяет указывать зависимости прямо в файле.

## Создание скрипта

```bash
# Создать скрипт с зависимостями
uv init --script script.py

# Добавить зависимости
uv add --script script.py requests
```

## Формат скрипта

```python
# /// script
# requires-python = ">=3.12"
# dependencies = [
#     "requests>=2.31",
#     "rich>=13.0",
# ]
# ///

import requests
from rich import print

response = requests.get("https://api.github.com")
print(f"Status: {response.status_code}")
```

## Запуск скрипта

```bash
# Запуск — uv автоматически создаст окружение и установит зависимости
uv run script.py

# С конкретным Python
uv run --python 3.12 script.py

# С дополнительными пакетами
uv run --with httpx script.py

# Запуск как модуль
uv run --module script
```

## Редактирование зависимостей

```bash
# Добавить зависимость в скрипт
uv add --script script.py httpx

# Удалить зависимость
uv remove --script script.py requests

# Показать зависимости
uv tree --script script.py

# Синхронизировать окружение
uv sync --script script.py
```

## Автоматическое обнаружение

uv автоматически обнаруживает inline метаданные при запуске:

```bash
# Оба варианта эквивалентны
uv run script.py
uv run --script script.py
```

## Best Practices

✅ **Используйте** для однофайловых утилит и скриптов
✅ **Указывайте** `requires-python` для воспроизводимости
✅ **Фиксируйте** версии зависимостей для стабильности

❌ **Не используйте** для больших проектов — выбирайте `uv init`

## Ссылки

- [PEP 723](https://peps.python.org/pep-0723/)
- [uv Scripts](https://docs.astral.sh/uv/guides/scripts/)

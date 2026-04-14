# uv — Управление версиями Python

## Обзор

uv может управлять версиями Python — скачивать, устанавливать, переключать между ними. Поддерживает CPython, PyPy, GraalPy.

## Форматы запроса Python

```bash
# Версия
3, 3.12, 3.12.3

# Спецификатор
">=3.12,<3.13"

# Реализация
cpython, cp, pypy

# Реализация + версия
cpython@3.12, cpython3.12, cp312

# Полная спецификация
cpython-3.12.3-macos-aarch64-none

# Путь к интерпретатору
/opt/homebrew/bin/python3
mypython3
```

## Команды

```bash
# Список доступных версий
uv python list

# Список установленных версий
uv python list --installed

# Установка Python
uv python install 3.12
uv python install 3.11 3.12 3.13
uv python install cpython@3.12
uv python install pypy@3.10

# Поиск конкретного интерпретатора
uv python find
uv python find 3.12
uv python find cpython@3.12

# Фиксация версии для проекта
uv python pin 3.12
uv python pin cpython@3.12

# Показать директорию установки
uv python dir

# Обновление установленных версий
uv python upgrade
uv python upgrade 3.12

# Удаление версий
uv python uninstall 3.12
uv python uninstall --all

# Обновление PATH
uv python update-shell
```

## .python-version

Файл `.python-version` фиксирует требуемую версию Python для проекта.

```
# .python-version
3.12
```

uv автоматически скачает и использует эту версию при запуске команд.

## Приоритет поиска Python

1. Активный виртуальный окружение (`VIRTUAL_ENV`)
2. `.venv` в текущей/родительских директориях
3. uv-managed Python (из `uv python install`)
4. Системный Python (из `PATH`)
5. Windows: реестр

## Флаги управления Python

```bash
# Только uv-managed Python
uv sync --managed-python

# Только системный Python
uv sync --no-managed-python

# Отключить автозагрузку
uv sync --no-python-downloads

# Конкретный интерпретатор
uv run --python 3.12 python main.py
uv run --python /opt/homebrew/bin/python3 main.py
uv run --python cpython@3.12 main.py
```

## Множественные платформы в lock

uv.lock поддерживает несколько платформ и версий Python одновременно:

```bash
# Lock для нескольких платформ
uv lock --python-platform linux --python-platform macos --python-platform windows
```

## Best Practices

✅ **Используйте** `.python-version` для фиксации версии
✅ **Коммитьте** `.python-version` в репозиторий
✅ **Используйте** `--managed-python` в CI для воспроизводимости
✅ **Указывайте** `requires-python` в `pyproject.toml`

❌ **Не удаляйте** системный Python через uv
❌ **Не используйте** разные версии Python в dev/prod

## Ссылки

- [uv Python Management](https://docs.astral.sh/uv/guides/install-python/)

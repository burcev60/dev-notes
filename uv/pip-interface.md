# uv — pip-совместимый интерфейс

## Описание

`uv pip` — drop-in замена pip с совместимым CLI но в 10-100 раз быстрее.

## Установка пакетов

```bash
# Как pip
uv pip install requests
uv pip install "requests>=2.28"
uv pip install -r requirements.txt

# Из файла
uv pip install -e .  # Editable install
uv pip install -e ./local-pkg

# Несколько пакетов
uv pip install fastapi pydantic uvicorn

# С ограничениями
uv pip install -c constraints.txt package

# С override
uv pip install --override overrides.txt package
```

## Создание окружения

```bash
# Создание venv
uv venv
uv venv .venv
uv venv --python 3.12
uv venv --python cpython@3.12
uv venv --seed  # С предустановленным pip

# С конкретным интерпретатором
uv venv --python /opt/homebrew/bin/python3
```

## Синхронизация

```bash
# Синхронизация с requirements.txt
uv pip sync requirements.txt

# С несколькими файлами
uv pip sync requirements-dev.txt requirements.txt

# Dry run
uv pip sync --dry-run requirements.txt

# Без удаления лишнего
uv pip sync --inexact requirements.txt
```

## Заморозка

```bash
# Freeze как pip
uv pip freeze > requirements.txt

# С аннотациями
uv pip freeze --exclude-editable
```

## Список пакетов

```bash
uv pip list
uv pip list --outdated
uv pip list --format json
```

## Удаление

```bash
uv pip uninstall requests
uv pip uninstall -r requirements.txt
uv pip uninstall --all
```

## Компиляция требований (pip-compile аналог)

```bash
# Компиляция в requirements.txt
uv pip compile requirements.in > requirements.txt

# С ограничениями
uv pip compile --constraint constraints.txt requirements.in

# С hash
uv pip compile --generate-hashes requirements.in

# Для конкретной платформы
uv pip compile --python-platform linux requirements.in

# С extras
uv pip compile --extra dev requirements.in

# Без dev
uv pip compile --no-deps requirements.in
```

## Отличия от pip

| Фича | pip | uv pip |
|------|-----|--------|
| Скорость | Медленный | 10-100x быстрее |
| Кэш | Локальный venv | Глобальный кэш |
| Установка | Copy | Hardlink/clone |
| Разрешение | Медленное | Быстрое (Rust) |
| pip compile | pip-tools | Встроено |

## Когда использовать uv pip

✅ **Миграция** с pip без изменения workflow
✅ **CI/CD** — быстрая установка зависимостей
✅ **Docker** — уменьшение времени сборки
✅ **Drop-in замена** — тот же CLI интерфейс

## Когда использовать uv project

✅ **Новые проекты** — `uv init`
✅ **Управление зависимостями** — `uv add/sync`
✅ **Lock-файлы** — `uv lock`
✅ **Workspaces** — монорепозитории

## Best Practices

✅ **Используйте** `uv pip compile` для генерации requirements.txt
✅ **Используйте** `--generate-hashes` для безопасности
✅ **Используйте** `uv venv` вместо `python -m venv`
✅ **Переключайтесь** на `uv project` для новых проектов

❌ **Не смешивайте** uv pip и uv project в одном проекте
❌ **Не используйте** `uv pip install` если есть pyproject.toml

## Ссылки

- [uv pip Interface](https://docs.astral.sh/uv/pip/)

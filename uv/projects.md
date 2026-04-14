# uv — Управление проектами

## Создание проекта

```bash
# Базовый проект
uv init my-project
cd my-project

# Приложение
uv init --app my-app

# Библиотека
uv init --lib my-lib

# С Git (VCS)
uv init --vcs git my-project

# Только pyproject.toml
uv init --bare

# С конкретным build backend
uv init --build-backend hatch
uv init --build-backend pdm
uv init --build-backend setuptools
uv init --build-backend uv
```

## pyproject.toml

```toml
[project]
name = "my-project"
version = "0.1.0"
description = "My project description"
requires-python = ">=3.12"
dependencies = [
    "fastapi>=0.110.0",
    "pydantic>=2.0",
]

[dependency-groups]
dev = [
    "pytest>=8.0",
    "ruff>=0.4.0",
    "mypy>=1.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

## Добавление зависимостей

```bash
# Основная зависимость
uv add fastapi
uv add "fastapi>=0.110.0"
uv add "pydantic>=2.0,<3.0"

# Dev зависимость
uv add --dev pytest
uv add --dev ruff

# Группа зависимостей
uv add --group test pytest pytest-asyncio
uv add --group docs mkdocs

# Optional dependency (extras)
uv add --optional uvicorn  # Для extra="uvicorn"

# Несколько пакетов
uv add fastapi pydantic sqlalchemy

# Из файла требований
uv add -r requirements.txt
```

## Удаление зависимостей

```bash
uv remove fastapi
uv remove --dev pytest
uv remove --group test pytest
```

## Sync — Синхронизация окружения

```bash
# Базовая синхронизация
uv sync

# С dev зависимостями (по умолчанию включены)
uv sync --dev

# Без dev зависимостей (для продакшена)
uv sync --no-dev

# Конкретная группа
uv sync --group test
uv sync --group test --group docs

# Только группа (без default)
uv sync --only-group test

# Все группы
uv sync --all-groups

# Точная синхронизация (удалить лишнее)
uv sync --exact

# Без установки проекта
uv sync --no-install-project

# Проверка без изменений
uv sync --dry-run

# Проверка синхронизации
uv sync --check

# Frozen — без обновления lock
uv sync --frozen

# Locked — проверить что lock не изменился
uv sync --locked

# Альтернативный Python интерпретатор
uv sync --python 3.11
```

## Lock — Фиксация зависимостей

```bash
# Обновить lock-файл
uv lock

# Обновить конкретный пакет
uv lock --upgrade-package fastapi

# Обновить все
uv lock --upgrade

# Обновить группу
uv lock --upgrade-group dev

# Frozen — без изменений
uv lock --frozen

# Для конкретной платформы
uv lock --python-platform linux
```

## Запуск команд

```bash
# Запуск Python скрипта
uv run python main.py

# Запуск модуля
uv run -m http.server

# Запуск с временными пакетами
uv run --with httpx python script.py
uv run --with-editable ./local-pkg python script.py

# Запуск с файлом требований
uv run --with-requirements requirements.txt python script.py

# Запуск команды из проекта
uv run pytest
uv run ruff check .
uv run mypy src/

# С конкретным extra
uv run --extra uvicorn uvicorn main:app

# Без dev зависимостей
uv run --no-dev python main.py

# С конкретной группой
uv run --group test pytest

# Только группа
uv run --only-group test pytest

# Загрузить .env файл
uv run --env-file .env python main.py

# Без .env файла
uv run --no-env-file python main.py

# Изолированный запуск (игнорировать проект)
uv run --isolated python script.py

# Использовать активный venv вместо проекта
uv run --active python script.py

# Без синхронизации
uv run --no-sync python script.py

# С точной синхронизацией
uv run --exact python main.py
```

## Дерево зависимостей

```bash
# Показать дерево
uv tree

# Глубина
uv tree --depth 2

# Обратные зависимости
uv tree --invert fastapi

# Обновлённые версии
uv tree --outdated

# Размеры колёс
uv tree --show-sizes

# Без дублирования
uv tree --no-dedupe

# Universal (все платформы)
uv tree --universal

# Для конкретной платформы
uv tree --python-platform linux

# Без dev зависимостей
uv tree --no-dev

# Конкретная группа
uv tree --group test

# Отфильтровать пакет
uv tree --prune setuptools
```

## Экспорт

```bash
# В requirements.txt
uv export --format requirements.txt

# В pylock.toml
uv export --format pylock.toml

# В CycloneDX 1.5 (SBOM)
uv export --format cyclonedx1.5

# С аннотациями (по умолчанию)
uv export --no-annotate

# Без хэшей
uv export --no-hashes

# В файл
uv export -o requirements.txt

# Без dev зависимостей
uv export --no-dev

# Конкретная группа
uv export --group test

# Без проекта
uv export --no-emit-project

# Workspace
uv export --all-packages
uv export --package my-subproject
```

## Версия проекта

```bash
# Показать версию
uv version

# Обновить
uv version patch   # 0.1.0 → 0.1.1
uv version minor   # 0.1.0 → 0.2.0
uv version major   # 0.1.0 → 1.0.0
uv version 1.0.0   # Конкретная версия
```

## Best Practices

✅ **Коммитьте** `uv.lock` в репозиторий
✅ **Используйте** `--frozen` в CI/CD
✅ **Используйте** `--no-dev` для продакшен образов
✅ **Используйте** `uv sync --check` валидацию
✅ **Добавляйте** `.venv/` в `.gitignore`
✅ **Используйте** dependency groups вместо extras для dev

❌ **Не игнорируйте** `uv.lock` в VCS
❌ **Не используйте** `--no-config` без необходимости
❌ **Не смешивайте** uv и pip в одном проекте

## Ссылки

- [uv Projects](https://docs.astral.sh/uv/guides/projects/)
- [uv Lock](https://docs.astral.sh/uv/guides/lock/)

# uv — Workspaces (Монорепозитории)

## Описание

Workspaces позволяют управлять несколькими пакетами в одном репозитории с общим lock-файлом.

## Структура workspace

```
my-workspace/
├── pyproject.toml       # Корневой workspace
├── uv.lock              # Общий lock-файл
├── .python-version
├── packages/
│   ├── core/
│   │   ├── pyproject.toml
│   │   └── src/core/
│   ├── api/
│   │   ├── pyproject.toml
│   │   └── src/api/
│   └── cli/
│       ├── pyproject.toml
│       └── src/cli/
```

## Корневой pyproject.toml

```toml
[tool.uv.workspace]
members = [
    "packages/core",
    "packages/api",
    "packages/cli",
]

# Исключения
# exclude = ["packages/experimental"]

[tool.uv.sources]
# Переопределение источников для workspace
# core = { workspace = true }
```

## Пакет в workspace

```toml
# packages/api/pyproject.toml
[project]
name = "my-api"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "my-core",         # Ссылка на другой пакет workspace
    "fastapi>=0.110",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

## Зависимости между пакетами

```bash
# Добавление зависимости на пакет workspace
uv add my-core
# uv автоматически использует workspace = true

# Добавление с конкретным пакетом
uv add --package my-api my-core
```

## Команды workspace

```bash
# Синхронизация всего workspace
uv sync

# Синхронизация конкретного пакета
uv sync --package my-api

# Все пакеты
uv sync --all-packages

# Запуск команды в конкретном пакете
uv run --package my-api python -m my_api

# Экспорт зависимостей пакета
uv export --package my-api

# Дерево зависимостей пакета
uv tree --package my-api

# Добавить зависимость конкретному пакету
uv add --package my-cli httpx
```

## Исключения

```toml
[tool.uv.workspace]
members = ["packages/*"]
exclude = ["packages/experimental"]
```

## Dev зависимости в workspace

Dev зависимости определяются в каждом пакете отдельно:

```toml
# packages/core/pyproject.toml
[dependency-groups]
dev = ["pytest", "mypy"]

# packages/api/pyproject.toml
[dependency-groups]
dev = ["pytest", "httpx"]
```

## Best Practices

✅ **Используйте** workspace для монорепозиториев
✅ **Определяйте** dev зависимости в каждом пакете
✅ **Используйте** `--package` для работы с конкретным пакетом
✅ **Коммитьте** общий `uv.lock`

❌ **Не создавайте** отдельные lock-файлы для пакетов
❌ **Не смешивайте** workspace и отдельные проекты

## Ссылки

- [uv Workspaces](https://docs.astral.sh/uv/concepts/workspaces/)

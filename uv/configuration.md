# uv — Конфигурация

## Файлы конфигурации

uv читает конфигурацию из нескольких мест (по приоритету):

1. Командная строка (высший приоритет)
2. Переменные окружения
3. `pyproject.toml` (в `[tool.uv]`)
4. `uv.toml` (в проекте или родительских директориях)
5. Глобальный `uv.toml`:
   - Linux: `$XDG_CONFIG_HOME/uv/uv.toml` или `~/.config/uv/uv.toml`
   - macOS: `~/Library/Application Support/uv/uv.toml`
   - Windows: `%APPDATA%\uv\uv.toml`

## uv.toml

```toml
# uv.toml

# Индекс пакетов
index-url = "https://pypi.org/simple"
extra-index-url = ["https://pypi.company.com/simple/"]

# Кэш
cache-dir = "/path/to/cache"

# Python
python-preference = "managed"  # managed | system

# Разрешение зависимостей
resolution = "highest"  # highest | lowest | lowest-direct
prerelease = "disallow"  # disallow | allow | if-necessary | explicit

# Исключить новые пакеты
exclude-newer = "2024-01-01"

# Форк стратегия
fork-strategy = "fewest"  # fewest | requires-python

# Link mode
link-mode = "clone"  # clone | copy | hardlink | symlink

# Компиляция bytecode
compile-bytecode = true
```

## pyproject.toml

```toml
[tool.uv]
# Index
index-url = "https://pypi.org/simple"

# Cache
cache-dir = "/path/to/cache"

# Python
python-preference = "managed"

# Разрешение
resolution = "highest"
prerelease = "disallow"
exclude-newer = "2024-01-01"
fork-strategy = "fewest"

# Build
no-build = false
no-binary = false
no-build-isolation = false

# Link
link-mode = "clone"

# Compile
compile-bytecode = false

# Sources
# ... (см. sources.md)

# Workspace
# ... (см. workspaces.md)

# Dev dependency groups по умолчанию
default-groups = ["dev", "test"]
```

## Переменные окружения

```bash
# Индекс
UV_INDEX_URL=https://pypi.org/simple
UV_EXTRA_INDEX_URL=https://pypi.company.com/simple/
UV_DEFAULT_INDEX=https://pypi.org/simple

# Python
UV_PYTHON=3.12
UV_MANAGED_PYTHON=true
UV_PYTHON_DOWNLOADS=never

# Кэш
UV_CACHE_DIR=/path/to/cache
UV_NO_CACHE=true

# Конфигурация
UV_CONFIG_FILE=/path/to/uv.toml
UV_NO_CONFIG=true

# Сеть
UV_OFFLINE=true
UV_INSECURE_HOST=localhost
UV_SYSTEM_CERTS=true

# Прогресс
UV_NO_PROGRESS=true

# Проект
UV_PROJECT=/path/to/project
UV_WORKING_DIR=/path/to/dir

# Зависимости
UV_CONSTRAINT=requirements.txt
UV_OVERRIDE=overrides.txt

# Build
UV_NO_BUILD=true
UV_NO_BINARY=true
UV_NO_BUILD_ISOLATION=true

# Resolution
UV_RESOLUTION=highest
UV_PRERELEASE=disallow
UV_EXCLUDE_NEWER=2024-01-01
UV_FORK_STRATEGY=fewest

# Install
UV_LINK_MODE=clone
UV_COMPILE_BYTECODE=true

# Env file
UV_ENV_FILE=.env
UV_NO_ENV_FILE=true
```

## .python-version

```
# Фиксация версии Python
3.12
```

## .env файл

uv автоматически загружает `.env` файл при `uv run`.

```env
# .env
DATABASE_URL=postgresql://localhost/db
SECRET_KEY=my-secret
DEBUG=true
```

```bash
# Запуск с .env (по умолчанию)
uv run python main.py

# С конкретным файлом
uv run --env-file .env.production python main.py

# Без .env
uv run --no-env-file python main.py
```

## Constraint файлы

Ограничения версий для всех зависимостей:

```txt
# constraints.txt
urllib3<2.0
requests>=2.28,<3.0
```

```bash
uv add --constraints constraints.txt package
```

## Override файлы

Переопределение версий:

```txt
# overrides.txt
requests==2.31.0
```

```bash
uv add --overrides overrides.txt package
```

## Best Practices

✅ **Используйте** `.python-version` для фиксации Python
✅ **Используйте** `exclude-newer` для воспроизводимости
✅ **Используйте** `.env` для локальных секретов
✅ **Коммитьте** `uv.toml` если нужна общая конфигурация

❌ **Не коммитьте** `.env` с секретами
❌ **Не используйте** `UV_INSECURE_HOST` в продакшене

## Ссылки

- [uv Configuration](https://docs.astral.sh/uv/concepts/configuration/)

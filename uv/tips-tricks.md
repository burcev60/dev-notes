# uv — Советы и трюки

## Миграция с pip

```bash
# 1. Создать проект
uv init my-project
cd my-project

# 2. Установить зависимости из requirements.txt
uv add -r requirements.txt

# 3. Сгенерировать lock
uv lock

# 4. Синхронизировать
uv sync

# 5. Удалить старый venv и requirements.txt
rm -rf venv requirements.txt
```

## Миграция с poetry

```bash
# poetry.lock → uv.lock
uv lock

# pyproject.toml совместим — uv читает [project] секцию
# Убедитесь что dependencies в [project], а не [tool.poetry]
```

## Миграция с pipenv

```bash
# Pipfile.lock → uv.lock
uv add -r requirements.txt  # Экспортируйте из Pipfile
uv lock
```

## Docker оптимизация

```dockerfile
FROM python:3.12-slim

# Установка uv
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv

WORKDIR /app

# Копирование зависимостей
COPY pyproject.toml uv.lock ./

# Установка без dev зависимостей
RUN uv sync --frozen --no-dev --no-install-project

# Копирование кода
COPY . .

# Установка проекта
RUN uv sync --frozen --no-dev

CMD ["uv", "run", "uvicorn", "main:app", "--host", "0.0.0.0"]
```

## CI/CD оптимизация

```yaml
# GitHub Actions
- uses: astral-sh/setup-uv@v4
  with:
    version: "latest"

- name: Install dependencies
  run: uv sync --frozen --no-dev

- name: Run tests
  run: uv run pytest
```

## Кэш в CI

```yaml
- name: Cache uv
  uses: actions/cache@v4
  with:
    path: ~/.cache/uv
    key: ${{ runner.os }}-uv-${{ hashFiles('uv.lock') }}
    restore-keys: |
      ${{ runner.os }}-uv-
```

## Быстрый запуск без проекта

```bash
# Запуск скрипта с зависимостями
uv run --with requests python -c "import requests; print(requests.get('https://example.com'))"

# Запуск CLI утилиты
uvx ruff check .  # uvx = uv tool run
uvx httpie https://example.com
```

## Debug и логирование

```bash
# Подробный вывод
uv sync -v
uv sync -vv
uv sync -vvv

# Тихий режим
uv sync -q
uv sync -qq  # Полная тишина

# Без цветов
uv sync --color never

# Без прогресса
uv sync --no-progress
```

## Оффлайн режим

```bash
# Работа без сети (только кэш)
uv sync --offline

# Разрешить конкретный хост
uv sync --allow-insecure-host localhost:8080
```

## Воспроизводимость

```bash
# Фиксация даты — только пакеты до этой даты
uv lock --exclude-newer 2024-01-01

# Lowest версии (для тестирования совместимости)
uv lock --resolution lowest

# Locked — проверка что lock не изменился
uv sync --locked
```

## Торч (PyTorch)

```bash
# С конкретным backend
uv add --torch-backend cu126 torch

# Автоопределение
uv add --torch-backend auto torch
```

## Aliases

```bash
# .bashrc / .zshrc
alias uvr="uv run"
alias uva="uv add"
alias uvs="uv sync"
alias uvl="uv lock"
alias uvt="uv tree"
alias uvx="uv tool run"
```

## Troubleshooting

```bash
# Очистить кэш
uv cache clean

# Удалить неиспользуемое
uv cache prune

# Переустановить все
uv sync --reinstall

# Обновить конкретный пакет
uv lock --upgrade-package fastapi

# Показать расположение кэша
uv cache dir

# Показать Python установки
uv python dir

# Проверить синхронизацию
uv sync --check
```

## Сравнение скорости

```bash
# pip
time pip install -r requirements.txt
# ~30s

# uv pip
time uv pip install -r requirements.txt
# ~2s

# uv project
time uv sync
# ~1s
```

## Best Practices

✅ **Используйте** `--frozen` в CI/CD
✅ **Кэшируйте** `~/.cache/uv` в CI
✅ **Используйте** `uvx` для быстрого запуска утилит
✅ **Используйте** `--exclude-newer` для воспроизводимости
✅ **Коммитьте** `uv.lock` и `.python-version`
✅ **Используйте** `-v` для отладки

❌ **Не используйте** `uv pip` вместе с `uv project`
❌ **Не игнорируйте** `uv.lock` в VCS
❌ **Не используйте** `--offline` без заполненного кэша

## Ссылки

- [uv Tips & Tricks](https://docs.astral.sh/uv/)
- [setup-uv GitHub Action](https://github.com/astral-sh/setup-uv)

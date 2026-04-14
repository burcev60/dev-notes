# uv — Обзор и установка

## Описание

**uv** — чрезвычайно быстрый менеджер пакетов Python, написанный на Rust. Разработан командой Astral (создатели Ruff).

**Версия:** uv 0.11.6

## Установка

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# pip
pip install uv

# Homebrew
brew install uv

# pipx
pipx install uv
```

## Проверка

```bash
uv --version
# uv 0.11.6

uv --help
```

## Почему uv?

| Фича | Описание |
|------|----------|
| **Скорость** | В 10-100 раз быстрее pip благодаря Rust и кэшированию |
| **Единый инструмент** | Заменяет pip, pip-tools, venv, pipx, pyenv |
| **Lock-файлы** | Детерминированные, кроссплатформенные (как Cargo.lock) |
| **Управление Python** | Автоматическая установка нужных версий Python |
| **Workspaces** | Монорепозитории из коробки |
| **PEP 723** | Скрипты с inline зависимостями |
| **pip-совместимость** | `uv pip install` как drop-in замена |
| **Кэш** | Глобальный кэш с hardlink/clone, экономия диска |

## Основные команды

```bash
# Проект
uv init              # Создать проект
uv add <pkg>         # Добавить зависимость
uv remove <pkg>      # Удалить зависимость
uv sync              # Синхронизировать окружение
uv lock              # Обновить lock-файл
uv run <cmd>         # Запустить команду в окружении
uv tree              # Дерево зависимостей
uv export            # Экспорт в requirements.txt

# Python
uv python list       # Доступные версии Python
uv python install 3.12  # Установить Python
uv python pin 3.12   # Зафиксировать версию

# Инструменты
uv tool install ruff  # Установить утилиту
uv tool run ruff      # Запустить утилиту

# Скрипты
uv run script.py      # Запустить скрипт с зависимостями
```

## Ключевые концепции

### Проект (Project)

Проект — директория с `pyproject.toml`. uv автоматически создаёт виртуальное окружение `.venv`.

```
my-project/
├── pyproject.toml   # Зависимости и метаданные
├── uv.lock          # Зафиксированные версии (детерминизм)
├── .python-version  # Требуемая версия Python
└── .venv/           # Виртуальное окружение (в .gitignore)
```

### Lock-файл

`uv.lock` — детерминированный lock-файл, гарантирует одинаковые версии зависимостей на всех машинах. Поддерживает несколько платформ и версий Python одновременно.

### Виртуальное окружение

uv автоматически создаёт `.venv` в корне проекта. Не нужно активировать — `uv run` использует его автоматически.

### Кэш

uv использует глобальный кэш (`~/.cache/uv` на Linux) для скачанных пакетов и версий Python. Пакеты устанавливаются через hardlink/clone, что экономит место на диске.

```bash
uv cache dir          # Показать путь к кэшу
uv cache clean        # Очистить кэш
uv cache prune        # Удалить неиспользуемое
```

## Обновление uv

```bash
uv self update        # Обновить до последней версии
uv self update 0.11.0 # Обновить до конкретной версии
```

## Ссылки

- [Официальная документация](https://docs.astral.sh/uv/)
- [GitHub репозиторий](https://github.com/astral-sh/uv)
- [Блог Astral](https://astral.sh/blog)

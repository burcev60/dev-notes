# 📚 Developer Notes

Заметки по Python, фреймворкам, библиотекам и смежным темам. Подробные гайды с примерами кода, best practices и ссылками на документацию.

---

## 📂 Навигация

### 🐍 Python

Раздел: [`python/`](python/README.md)

| Тема | Файл |
|------|------|
| **Стандартные библиотеки** | |
| · os, os.path | [`python/std-libraries/os-path.md`](python/std-libraries/os-path.md) |
| · sys | [`python/std-libraries/sys.md`](python/std-libraries/sys.md) |
| · collections | [`python/std-libraries/collections.md`](python/std-libraries/collections.md) |
| · itertools | [`python/std-libraries/itertools.md`](python/std-libraries/itertools.md) |
| · datetime | [`python/std-libraries/datetime.md`](python/std-libraries/datetime.md) |
| · json | [`python/std-libraries/json.md`](python/std-libraries/json.md) |
| · functools | [`python/std-libraries/functools.md`](python/std-libraries/functools.md) |
| · typing | [`python/std-libraries/typing-module.md`](python/std-libraries/typing-module.md) |
| · logging | [`python/std-libraries/logging.md`](python/std-libraries/logging.md) |
| · re | [`python/std-libraries/re.md`](python/std-libraries/re.md) |
| · contextlib | [`python/std-libraries/contextlib.md`](python/std-libraries/contextlib.md) |
| **Встроенные функции** | [`python/built-ins/functions.md`](python/built-ins/functions.md) |
| **Типизация** | |
| · Основы | [`python/type-hints/basics.md`](python/type-hints/basics.md) |
| · Дженерики | [`python/type-hints/generics.md`](python/type-hints/generics.md) |
| · Протоколы | [`python/type-hints/protocols.md`](python/type-hints/protocols.md) |
| **ООП и паттерны** | |
| · Классы и наследование | [`python/oop-patterns/classes.md`](python/oop-patterns/classes.md) |
| · Декораторы | [`python/oop-patterns/decorators.md`](python/oop-patterns/decorators.md) |
| · Итераторы и генераторы | [`python/oop-patterns/iterators-generators.md`](python/oop-patterns/iterators-generators.md) |
| · Dataclasses | [`python/oop-patterns/dataclasses.md`](python/oop-patterns/dataclasses.md) |
| **Асинхронность** | |
| · Основы asyncio | [`python/async/asyncio-basics.md`](python/async/asyncio-basics.md) |
| · async/await паттерны | [`python/async/async-await.md`](python/async/async-await.md) |
| · Конкурентное выполнение | [`python/async/concurrent-execution.md`](python/async/concurrent-execution.md) |
| · Синхронизация | [`python/async/synchronization.md`](python/async/synchronization.md) |
| · Обработка ошибок | [`python/async/error-handling.md`](python/async/error-handling.md) |
| · HTTP клиенты | [`python/async/http-clients.md`](python/async/http-clients.md) |
| · Тестирование | [`python/async/testing.md`](python/async/testing.md) |
| · Частые ошибки | [`python/async/common-mistakes.md`](python/async/common-mistakes.md) |

---

### ⚡ FastAPI

Раздел: [`fastapi/`](fastapi/README.md)

| Тема | Файл |
|------|------|
| Основы: роутинг, зависимости, DI | [`fastapi/basics.md`](fastapi/basics.md) |
| Pydantic модели и валидация | [`fastapi/pydantic-models.md`](fastapi/pydantic-models.md) |
| Система зависимостей | [`fastapi/dependency-injection.md`](fastapi/dependency-injection.md) |
| Middleware и хуки | [`fastapi/middleware.md`](fastapi/middleware.md) |
| WebSocket | [`fastapi/websockets.md`](fastapi/websockets.md) |
| Тестирование | [`fastapi/testing.md`](fastapi/testing.md) |
| Деплой (Uvicorn, Gunicorn) | [`fastapi/deployment.md`](fastapi/deployment.md) |
| Best practices | [`fastapi/best-practices.md`](fastapi/best-practices.md) |

#### Продвинутые темы

| Тема | Файл |
|------|------|
| Аутентификация: OAuth2, JWT | [`fastapi/auth.md`](fastapi/auth.md) |
| Интеграция с БД (SQLAlchemy async) | [`fastapi/database.md`](fastapi/database.md) |
| Структура проекта | [`fastapi/project-structure.md`](fastapi/project-structure.md) |
| Продвинутое тестирование | [`fastapi/advanced-testing.md`](fastapi/advanced-testing.md) |
| Логирование и мониторинг | [`fastapi/logging-monitoring.md`](fastapi/logging-monitoring.md) |
| Кэширование | [`fastapi/caching.md`](fastapi/caching.md) |
| Кастомизация документации | [`fastapi/custom-docs.md`](fastapi/custom-docs.md) |
| Background Tasks, Celery | [`fastapi/background-tasks.md`](fastapi/background-tasks.md) |
| Продвинутый middleware | [`fastapi/advanced-middleware.md`](fastapi/advanced-middleware.md) |

---

### 🗄️ SQLAlchemy

Раздел: [`sqlalchemy/`](sqlalchemy/README.md)

#### SQLAlchemy Core

| Тема | Файл |
|------|------|
| Engine и connection pool | [`sqlalchemy/core/engine.md`](sqlalchemy/core/engine.md) |
| SQL выражения | [`sqlalchemy/core/sql-expressions.md`](sqlalchemy/core/sql-expressions.md) |
| Транзакции | [`sqlalchemy/core/transactions.md`](sqlalchemy/core/transactions.md) |

#### SQLAlchemy ORM

| Тема | Файл |
|------|------|
| Declarative models | [`sqlalchemy/orm/models.md`](sqlalchemy/orm/models.md) |
| Relationships | [`sqlalchemy/orm/relationships.md`](sqlalchemy/orm/relationships.md) |
| Запросы (Query) | [`sqlalchemy/orm/querying.md`](sqlalchemy/orm/querying.md) |
| Sessions | [`sqlalchemy/orm/sessions.md`](sqlalchemy/orm/sessions.md) |
| Event listeners | [`sqlalchemy/orm/events.md`](sqlalchemy/orm/events.md) |

#### Alembic

| Тема | Файл |
|------|------|
| Миграции | [`sqlalchemy/alembic/migrations.md`](sqlalchemy/alembic/migrations.md) |
| Операции | [`sqlalchemy/alembic/operations.md`](sqlalchemy/alembic/operations.md) |

---

### ✅ Pydantic

Раздел: [`pydantic/`](pydantic/README.md)

| Тема | Файл |
|------|------|
| BaseModel и валидация | [`pydantic/models.md`](pydantic/models.md) |
| BaseSettings | [`pydantic/settings.md`](pydantic/settings.md) |
| Сериализация | [`pydantic/serialization.md`](pydantic/serialization.md) |
| Отличия v1 → v2 | [`pydantic/v2-changes.md`](pydantic/v2-changes.md) |

---

### 🤖 LangChain Ecosystem

Раздел: [`langchain-ecosystem/`](langchain-ecosystem/README.md)

#### LangChain

| Тема | Файл |
|------|------|
| Chains | [`langchain-ecosystem/langchain/chains.md`](langchain-ecosystem/langchain/chains.md) |
| Agents | [`langchain-ecosystem/langchain/agents.md`](langchain-ecosystem/langchain/agents.md) |
| Memory | [`langchain-ecosystem/langchain/memory.md`](langchain-ecosystem/langchain/memory.md) |
| Retrievers (RAG) | [`langchain-ecosystem/langchain/retrievers.md`](langchain-ecosystem/langchain/retrievers.md) |

#### LangGraph

| Тема | Файл |
|------|------|
| Основы StateGraph | [`langchain-ecosystem/langgraph/graph-basics.md`](langchain-ecosystem/langgraph/graph-basics.md) |
| Управление состоянием | [`langchain-ecosystem/langgraph/state.md`](langchain-ecosystem/langgraph/state.md) |
| Persistence | [`langchain-ecosystem/langgraph/persistence.md`](langchain-ecosystem/langgraph/persistence.md) |
| Human-in-the-loop | [`langchain-ecosystem/langgraph/human-in-loop.md`](langchain-ecosystem/langgraph/human-in-loop.md) |

#### Ollama

| Тема | Файл |
|------|------|
| Установка и запуск | [`langchain-ecosystem/ollama/setup.md`](langchain-ecosystem/ollama/setup.md) |
| Интеграция с LangChain | [`langchain-ecosystem/ollama/integration.md`](langchain-ecosystem/ollama/integration.md) |

#### Qdrant

| Тема | Файл |
|------|------|
| Подключение, CRUD | [`langchain-ecosystem/qdrant/client.md`](langchain-ecosystem/qdrant/client.md) |
| Векторный поиск | [`langchain-ecosystem/qdrant/vector-search.md`](langchain-ecosystem/qdrant/vector-search.md) |

---

### 🌐 HTTP Clients

Раздел: [`http-clients/`](http-clients/README.md)

| Библиотека | Файл |
|------------|------|
| requests (sync) | [`http-clients/requests.md`](http-clients/requests.md) |
| httpx (async) | [`http-clients/httpx.md`](http-clients/httpx.md) |
| aiohttp (async) | [`http-clients/aiohttp.md`](http-clients/aiohttp.md) |

---

### 🐳 Docker

Раздел: [`docker/`](docker/README.md)

| Тема | Файл |
|------|------|
| Основы: Dockerfile, образы | [`docker/basics.md`](docker/basics.md) |
| Docker Compose | [`docker/docker-compose.md`](docker/docker-compose.md) |
| Multi-stage builds | [`docker/multi-stage.md`](docker/multi-stage.md) |
| Python best practices | [`docker/python-best-practices.md`](docker/python-best-practices.md) |

---

### 🏗️ Архитектура

Раздел: [`architecture/`](architecture/README.md)

| Тема | Файл |
|------|------|
| SOLID принципы | [`architecture/solid.md`](architecture/solid.md) |
| Чистая архитектура | [`architecture/clean-architecture.md`](architecture/clean-architecture.md) |
| Dependency Injection | [`architecture/di-patterns.md`](architecture/di-patterns.md) |

#### Design Patterns

| Категория | Файл |
|-----------|------|
| Порождающие | [`architecture/design-patterns/creational.md`](architecture/design-patterns/creational.md) |
| Структурные | [`architecture/design-patterns/structural.md`](architecture/design-patterns/structural.md) |
| Поведенческие | [`architecture/design-patterns/behavioral.md`](architecture/design-patterns/behavioral.md) |

---

### 💾 Базы данных

Раздел: [`databases/`](databases/README.md)

#### PostgreSQL

| Тема | Файл |
|------|------|
| Основы | [`databases/postgresql/basics.md`](databases/postgresql/basics.md) |
| Индексы и оптимизация | [`databases/postgresql/indexes.md`](databases/postgresql/indexes.md) |
| Планы запросов (EXPLAIN) | [`databases/postgresql/query-plans.md`](databases/postgresql/query-plans.md) |

#### SQL

| Тема | Файл |
|------|------|
| JOIN типы | [`databases/sql/joins.md`](databases/sql/joins.md) |
| Оконные функции | [`databases/sql/window-functions.md`](databases/sql/window-functions.md) |
| CTE (Common Table Expressions) | [`databases/sql/cte.md`](databases/sql/cte.md) |

---

### 🦋 uv

Пакетный менеджер для Python от Astral. Невероятно быстрый, написан на Rust.

Раздел: [`uv/`](uv/README.md)

| Файл | Тема |
|------|------|
| Обзор и установка | [`uv/overview.md`](uv/overview.md) |
| Управление проектами | [`uv/projects.md`](uv/projects.md) |
| Управление Python | [`uv/python-management.md`](uv/python-management.md) |
| Tools (утилиты) | [`uv/tools.md`](uv/tools.md) |
| Скрипты (PEP 723) | [`uv/scripts.md`](uv/scripts.md) |
| Workspaces | [`uv/workspaces.md`](uv/workspaces.md) |
| Источники зависимостей | [`uv/sources.md`](uv/sources.md) |
| Конфигурация | [`uv/configuration.md`](uv/configuration.md) |
| pip-интерфейс | [`uv/pip-interface.md`](uv/pip-interface.md) |
| Советы и трюки | [`uv/tips-tricks.md`](uv/tips-tricks.md) |

---

## 📝 Формат заметок

Каждая заметка содержит:
- **Описание** — что делает и зачем нужно
- **Синтаксис** — сигнатуры и параметры
- **Примеры** — рабочий код с комментариями
- **Best practices** — рекомендации и антипаттерны
- **Ссылки** — на официальную документацию

# 🗄️ SQLAlchemy

Заметки по SQLAlchemy: Core, ORM и Alembic.

## 📂 Подразделы

| Раздел | Описание |
|--------|----------|
| [`core/`](core/README.md) | SQLAlchemy Core: Engine, SQL выражения, транзакции |
| [`orm/`](orm/README.md) | SQLAlchemy ORM: Declarative, Relationships, Sessions |
| [`alembic/`](alembic/README.md) | Alembic: миграции базы данных |

## 📂 Заметки

### Core

| Файл | Тема |
|------|------|
| [`core/engine.md`](core/engine.md) | Engine, connection pool |
| [`core/sql-expressions.md`](core/sql-expressions.md) | select, insert, update, delete |
| [`core/transactions.md`](core/transactions.md) | Транзакции, savepoints |

### ORM

| Файл | Тема |
|------|------|
| [`orm/models.md`](orm/models.md) | Declarative, маппинг моделей |
| [`orm/relationships.md`](orm/relationships.md) | one-to-many, many-to-many |
| [`orm/querying.md`](orm/querying.md) | Query, selectinload, joinedload |
| [`orm/sessions.md`](orm/sessions.md) | Session, sessionmaker |
| [`orm/events.md`](orm/events.md) | Event listeners |

### Alembic

| Файл | Тема |
|------|------|
| [`alembic/migrations.md`](alembic/migrations.md) | Создание и применение миграций |
| [`alembic/operations.md`](alembic/operations.md) | op.add_column и другие операции |

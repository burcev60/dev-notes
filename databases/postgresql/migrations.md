# PostgreSQL — Alembic: Миграции

## Описание

Alembic — инструмент управления миграциями для SQLAlchemy. Поддерживает авто-генерацию, ручные правки, ветвление.

## Установка и инициализация

```bash
pip install alembic

# Инициализация
alembic init alembic
```

## Структура

```
alembic/
├── env.py                 # Конфигурация окружения
├── README
├── script.py.mako         # Шаблон для новых миграций
└── versions/
    ├── 001_create_users.py
    ├── 002_create_posts.py
    └── 003_add_user_roles.py
alembic.ini                # Главный конфиг
```

## alembic.ini

```ini
[alembic]
script_location = alembic
sqlalchemy.url = postgresql+asyncpg://user:pass@localhost/mydb

# Формат ревизии
file_template = %%(year)d_%%(month).2d_%%(day).2d_%%(rev)s_%%(slug)s
```

## env.py (синхронный режим)

```python
# alembic/env.py
from logging.config import fileConfig
from sqlalchemy import engine_from_config, pool
from alembic import context
from app.models.base import Base
from app.models import user, post  # Импорт всех моделей!
from app.config import settings

config = context.config

if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = Base.metadata

def run_migrations_offline():
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )
    with context.begin_transaction():
        context.run_migrations()

def run_migrations_online():
    connectable = engine_from_config(
        config.get_section(config.config_ini_text),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )
    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata,
        )
        with context.begin_transaction():
            context.run_migrations()

if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

## env.py (асинхронный режим)

```python
# alembic/env.py
import asyncio
from logging.config import fileConfig
from sqlalchemy.ext.asyncio import create_async_engine
from alembic import context
from app.models.base import Base
from app.models import user, post
from app.config import settings

config = context.config

if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = Base.metadata

def run_migrations_offline():
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
    )
    with context.begin_transaction():
        context.run_migrations()

async def run_migrations_online():
    connectable = create_async_engine(settings.database_url)
    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)
    await connectable.dispose()

def do_run_migrations(connection):
    context.configure(connection=connection, target_metadata=target_metadata)
    with context.begin_transaction():
        context.run_migrations()

if context.is_offline_mode():
    run_migrations_offline()
else:
    asyncio.run(run_migrations_online())
```

## Авто-генерация миграций

```bash
# Создать миграцию на основе изменений в моделях
alembic revision --autogenerate -m "create users table"

# Применить миграцию
alembic upgrade head

# Проверить что миграция пустая (нет изменений)
alembic revision --autogenerate -m "empty"
# Если пусто — модели совпадают с БД
```

### Пример авто-сгенерированной миграции

```python
# alembic/versions/2024_01_15_abc123_create_users.py
"""create users table

Revision ID: abc123
Revises:
Create Date: 2024-01-15 10:30:00.000000

"""
from alembic import op
import sqlalchemy as sa

revision = 'abc123'
down_revision = None
branch_labels = None
depends_on = None


def upgrade() -> None:
    op.create_table(
        'users',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('username', sa.String(length=50), nullable=False),
        sa.Column('email', sa.String(length=100), nullable=False),
        sa.Column('hashed_password', sa.String(length=255), nullable=False),
        sa.Column('is_active', sa.Boolean(), nullable=True),
        sa.Column('created_at', sa.DateTime(timezone=True), server_default=sa.text('now()')),
        sa.PrimaryKeyConstraint('id'),
        sa.UniqueConstraint('email'),
        sa.UniqueConstraint('username'),
    )
    op.create_index('ix_users_email', 'users', ['email'])
    op.create_index('ix_users_username', 'users', ['username'])


def downgrade() -> None:
    op.drop_index('ix_users_username', table_name='users')
    op.drop_index('ix_users_email', table_name='users')
    op.drop_table('users')
```

## Ручные миграции

```bash
# Создать пустую миграцию
alembic revision -m "add user roles"
```

```python
# alembic/versions/2024_01_16_def456_add_user_roles.py
"""add user roles

Revision ID: def456
Revises: abc123
Create Date: 2024-01-16 10:30:00.000000

"""
from alembic import op
import sqlalchemy as sa

revision = 'def456'
down_revision = 'abc123'
branch_labels = None
depends_on = None


def upgrade() -> None:
    # Добавление колонки
    op.add_column('users', sa.Column('role', sa.String(50), server_default='user'))

    # Создание индекса
    op.create_index('ix_users_role', 'users', ['role'])

    # Добавление check constraint
    op.create_check_constraint(
        'valid_role',
        'users',
        "role IN ('admin', 'user', 'moderator')"
    )

    # Заполнение существующих данных
    op.execute("UPDATE users SET role = 'admin' WHERE username = 'admin'")


def downgrade() -> None:
    op.drop_constraint('valid_role', 'users', type_='check')
    op.drop_index('ix_users_role', table_name='users')
    op.drop_column('users', 'role')
```

## Операции Alembic

```python
# Таблицы
op.create_table('posts', ...)
op.drop_table('posts')
op.rename_table('old_name', 'new_name')

# Колонки
op.add_column('users', sa.Column('age', sa.Integer()))
op.drop_column('users', 'age')
op.alter_column('users', 'email', type_=sa.String(200))
op.alter_column('users', 'name', nullable=False)
op.rename_column('users', 'old_name', 'new_name')

# Индексы
op.create_index('ix_users_email', 'users', ['email'])
op.drop_index('ix_users_email', table_name='users')
op.create_index('ix_users_email', 'users', ['email'], unique=True)

# Foreign Keys
op.create_foreign_key('fk_posts_user_id', 'posts', 'users', ['user_id'], ['id'])
op.drop_constraint('fk_posts_user_id', 'posts', type_='foreignkey')

# Check Constraints
op.create_check_constraint('valid_age', 'users', 'age >= 0')
op.drop_constraint('valid_age', 'users', type_='check')

# Данные
op.execute("UPDATE users SET is_active = true WHERE is_active IS NULL")
```

## Управление миграциями

```bash
# Применить все миграции
alembic upgrade head

# Применить на N миграций
alembic upgrade +2

# Откатить на N миграций
alembic downgrade -1

# Откатить до конкретной ревизии
alembic downgrade abc123

# Откатить всё
alembic downgrade base

# Текущая ревизия
alembic current

# История миграций
alembic history
alembic history --verbose

# Показать SQL для миграции
alembic upgrade head --sql

# Проверить статус
alembic current
alembic heads
```

## Ветвление миграций

```bash
# Создать ветку
alembic revision --head=abc123 -m "branch feature"

# Объединение веток
alembic revision --merge abc123 def456 -m "merge branches"

# Проверка на ветвление
alembic heads  # Если > 1 — есть ветвление
```

## Best Practices

✅ **Используйте** `--autogenerate` для начальной миграции
✅ **Проверяйте** авто-сгенерированные миграции перед применением
✅ **Реализуйте** `downgrade()` для всех миграций
✅ **Коммитьте** миграции в VCS
✅ **Используйте** `op.execute()` для данных, `op.add_column()` для схемы
✅ **Тестируйте** миграции на staging перед продакшеном

❌ **Не изменяйте** применённые миграции
❌ **Не удаляйте** миграции из VCS
❌ **Не игнорируйте** `downgrade()`
❌ **Не запускайте** `upgrade head` без бэкапа на продакшене
❌ **Не забывайте** импортировать модели в `env.py`

## Ссылки

- [Alembic документация](https://alembic.sqlalchemy.org/)
- [Alembic Operations](https://alembic.sqlalchemy.org/en/latest/ops.html)
- [Alembic Async Cookbook](https://alembic.sqlalchemy.org/en/latest/cookbook.html#asyncio-support)

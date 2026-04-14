# Alembic — Миграции

## Описание

Alembic — инструмент для управления миграциями базы данных в SQLAlchemy.

## Инициализация

```bash
alembic init alembic
```

## Конфигурация

```python
# alembic.ini
sqlalchemy.url = postgresql://user:pass@localhost/db

# alembic/env.py
target_metadata = Base.metadata  # Из вашего приложения
```

## Создание миграции

```bash
# Автоматически из моделей
alembic revision --autogenerate -m "create users table"

# Вручную
alembic revision -m "add column to users"
```

## Применение

```bash
alembic upgrade head        # До последней
alembic upgrade +2          # На 2 миграции вперёд
alembic downgrade -1        # На 1 назад
```

## Best Practices

✅ **Используйте** `--autogenerate` для автосоздания
✅ **Проверяйте** сгенерированные миграции

## Ссылки

- [Alembic документация](https://alembic.sqlalchemy.org/)

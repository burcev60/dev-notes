# SQLAlchemy Core — Engine

## Описание

Engine — точка подключения к базе данных. Управляет connection pool и диалектом.

## Создание Engine

```python
from sqlalchemy import create_engine

# PostgreSQL
engine = create_engine(
    "postgresql+psycopg://user:password@localhost/dbname",
    echo=True,          # Логирование SQL
    pool_size=10,       # Размер пула
    max_overflow=20,    # Макс. превышение
    pool_timeout=30,    # Таймаут ожидания
    pool_recycle=1800,  # Переподключение (сек)
)

# SQLite
engine = create_engine("sqlite:///example.db")
```

## Connection

```python
# Прямое подключение
with engine.connect() as conn:
    result = conn.execute(text("SELECT 1"))
    row = result.fetchone()

# С транзакцией
with engine.begin() as conn:
    conn.execute(text("INSERT INTO users (name) VALUES (:name"), {"name": "Alice"})
    # Автоматический commit при выходе
```

## Best Practices

✅ **Используйте** `with engine.connect()` для автозакрытия
✅ **Настройте** `pool_size` под нагрузку
✅ **Используйте** `pool_recycle` для переподключения

## Ссылки

- [SQLAlchemy Engine](https://docs.sqlalchemy.org/en/20/core/connections.html)

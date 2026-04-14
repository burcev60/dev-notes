# SQLAlchemy — Транзакции

## Описание

Управление транзакциями в SQLAlchemy Core и ORM.

## Базовая транзакция

```python
from sqlalchemy import text

with engine.begin() as conn:
    conn.execute(text("INSERT INTO accounts (balance) VALUES (1000)"))
    conn.execute(text("UPDATE accounts SET balance = balance - 100 WHERE id = 1"))
    conn.execute(text("UPDATE accounts SET balance = balance + 100 WHERE id = 2"))
    # Автоматический commit при выходе
```

## Ручная транзакция

```python
with engine.connect() as conn:
    trans = conn.begin()
    try:
        conn.execute(text("INSERT INTO ..."))
        trans.commit()
    except:
        trans.rollback()
        raise
```

## Savepoint

```python
with engine.begin() as conn:
    conn.execute(text("INSERT INTO ..."))
    sp = conn.begin_nested()  # Savepoint
    try:
        conn.execute(text("INSERT INTO ..."))
        sp.commit()
    except:
        sp.rollback()
```

## Best Practices

✅ **Используйте** `begin()` для явной транзакции
✅ **Используйте** savepoints для частичного отката

## Ссылки

- [SQLAlchemy Transactions](https://docs.sqlalchemy.org/en/20/core/connections.html#working-with-transactions)

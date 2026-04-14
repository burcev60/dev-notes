# SQLAlchemy Core — SQL выражения

## Описание

SQLAlchemy Core предоставляет API для построения SQL запросов без ORM.

## Select

```python
from sqlalchemy import table, column, select

users = table('users', column('id'), column('name'), column('email'))

# Базовый SELECT
stmt = select(users.c.id, users.c.name).where(users.c.name == 'Alice')

# С условиями
stmt = select(users).where(
    (users.c.age >= 18) & (users.c.is_active == True)
)

# LIKE, IN
stmt = select(users).where(
    users.c.name.like('%Alice%')
)
stmt = select(users).where(
    users.c.id.in_([1, 2, 3])
)
```

## Insert, Update, Delete

```python
from sqlalchemy import insert, update, delete

# INSERT
stmt = insert(users).values(name='Bob', email='bob@example.com')

# INSERT с возвращаемым значением
stmt = insert(users).values(name='Charlie').returning(users.c.id)

# UPDATE
stmt = update(users).where(users.c.id == 1).values(name='Alice Updated')

# DELETE
stmt = delete(users).where(users.c.id == 1)
```

## Выполнение

```python
with engine.connect() as conn:
    # Один запрос
    result = conn.execute(stmt)
    conn.commit()

    # Несколько параметров
    conn.execute(
        insert(users),
        [
            {"name": "Alice", "email": "a@x.com"},
            {"name": "Bob", "email": "b@x.com"},
        ]
    )
    conn.commit()
```

## Best Practices

✅ **Используйте** Core для простых запросов
✅ **Используйте** `returning()` для получения ID

## Ссылки

- [SQLAlchemy Core](https://docs.sqlalchemy.org/en/20/core/dml.html)

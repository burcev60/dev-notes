# SQLAlchemy ORM — Запросы

## Описание

Запросы в SQLAlchemy ORM через Session.

## Базовые запросы

```python
# SELECT * FROM users
stmt = select(User)
users = session.execute(stmt).scalars().all()

# SELECT с условием
stmt = select(User).where(User.name == 'Alice')
user = session.execute(stmt).scalar_one_or_none()

# ORDER BY
stmt = select(User).order_by(User.name.desc())

# LIMIT, OFFSET
stmt = select(User).offset(10).limit(5)
```

## Добавление, обновление, удаление

```python
# INSERT
user = User(name="Bob", email="bob@example.com")
session.add(user)
session.commit()

# UPDATE
user = session.get(User, 1)
user.name = "Bob Updated"
session.commit()

# DELETE
session.delete(user)
session.commit()
```

## Best Practices

✅ **Используйте** `session.get()` для получения по ID
✅ **Используйте** `scalar_one_or_none()` для одного результата

## Ссылки

- [SQLAlchemy Querying](https://docs.sqlalchemy.org/en/20/orm/query.html)

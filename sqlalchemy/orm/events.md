# SQLAlchemy ORM — Events

## Описание

Event listeners для перехода операций с моделями.

## Основные события

```python
from sqlalchemy import event

@event.listens_for(User, 'before_insert')
def receive_before_insert(mapper, connection, target):
    print(f"About to insert: {target}")

@event.listens_for(User, 'after_insert')
def receive_after_insert(mapper, connection, target):
    print(f"Inserted: {target}")
```

## Session события

```python
@event.listens_for(Session, 'before_commit')
def receive_before_commit(session):
    print("About to commit")
```

## Best Practices

✅ **Используйте** events для логирования, аудита

## Ссылки

- [SQLAlchemy Events](https://docs.sqlalchemy.org/en/20/core/event.html)

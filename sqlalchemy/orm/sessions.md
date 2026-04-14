# SQLAlchemy ORM — Sessions

## Описание

Session — рабочий блок для ORM операций.

## Создание Session

```python
from sqlalchemy.orm import sessionmaker

SessionLocal = sessionmaker(bind=engine)

# Использование
with SessionLocal() as session:
    user = session.get(User, 1)
    session.add(User(name="Bob"))
    session.commit()
```

## Async Session (2.0)

```python
from sqlalchemy.ext.asyncio import async_sessionmaker, create_async_engine

engine = create_async_engine("postgresql+asyncpg://user:pass@localhost/db")
AsyncSession = async_sessionmaker(engine)

async with AsyncSession() as session:
    result = await session.execute(select(User))
    users = result.scalars().all()
    await session.commit()
```

## Best Practices

✅ **Используйте** `with Session()` для автозакрытия
✅ **Используйте** async session для async приложений

## Ссылки

- [SQLAlchemy Sessions](https://docs.sqlalchemy.org/en/20/orm/session.html)

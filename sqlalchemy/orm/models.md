# SQLAlchemy ORM — Модели

## Описание

Declarative — способ определения ORM моделей через классы.

## Declarative (SQLAlchemy 2.0)

```python
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from sqlalchemy import String, Integer

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = 'users'

    id: Mapped[int] = mapped_column(Integer, primary_key=True)
    name: Mapped[str] = mapped_column(String(50))
    email: Mapped[str] = mapped_column(String(100), unique=True)
    age: Mapped[int] = mapped_column(Integer, default=0)

    def __repr__(self):
        return f"User(id={self.id}, name={self.name!r})"
```

## Создание таблиц

```python
# Все таблицы
Base.metadata.create_all(engine)

# Отдельная таблица
User.__table__.create(engine)
```

## Конфигурация

```python
class User(Base):
    __tablename__ = 'users'
    __table_args__ = (
        UniqueConstraint('email'),
        Index('idx_user_name', 'name'),
    )
```

## Best Practices

✅ **Используйте** `Mapped` аннотации (SQLAlchemy 2.0)
✅ **Используйте** `DeclarativeBase` вместо `declarative_base()`

## Ссылки

- [SQLAlchemy Declarative](https://docs.sqlalchemy.org/en/20/orm/quickstart.html)

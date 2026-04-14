# SQLAlchemy ORM — Relationships

## Описание

Связи между моделями: one-to-many, many-to-one, many-to-many.

## One-to-Many

```python
class Parent(Base):
    __tablename__ = 'parents'
    id = mapped_column(Integer, primary_key=True)
    children = relationship("Child", back_populates="parent")

class Child(Base):
    __tablename__ = 'children'
    id = mapped_column(Integer, primary_key=True)
    parent_id = mapped_column(Integer, ForeignKey('parents.id'))
    parent = relationship("Parent", back_populates="children")
```

## Many-to-Many

```python
association_table = Table(
    'association', Base.metadata,
    Column('left_id', ForeignKey('left.id')),
    Column('right_id', ForeignKey('right.id'))
)

class Left(Base):
    __tablename__ = 'left'
    id = mapped_column(Integer, primary_key=True)
    rights = relationship("Right", secondary=association_table, back_populates="lefts")

class Right(Base):
    __tablename__ = 'right'
    id = mapped_column(Integer, primary_key=True)
    lefts = relationship("Left", secondary=association_table, back_populates="rights")
```

## Загрузка связей

```python
# selectinload — отдельный запрос для коллекции
stmt = select(User).options(selectinload(User.posts))

# joinedload — JOIN в одном запросе
stmt = select(User).options(joinedload(User.profile))

# subqueryload — подзапрос
stmt = select(User).options(subqueryload(User.comments))
```

## Best Practices

✅ **Используйте** `selectinload` для коллекций
✅ **Используйте** `joinedload` для one-to-one

## Ссылки

- [SQLAlchemy Relationships](https://docs.sqlalchemy.org/en/20/orm/relationships.html)

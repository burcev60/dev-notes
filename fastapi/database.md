# FastAPI — Интеграция с БД: SQLAlchemy async

## Описание

Интеграция SQLAlchemy с FastAPI через асинхронные сессии.

## Настройка движка

```python
# app/db/engine.py
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker
from app.config import settings

engine = create_async_engine(
    settings.database_url,
    echo=settings.debug,
    pool_size=settings.db_pool_size,
    max_overflow=settings.db_max_overflow,
    pool_recycle=1800,  # Переподключение каждые 30 мин
)

AsyncSession = async_sessionmaker(
    engine,
    expire_on_commit=False,  # Важно для async!
)
```

## Зависимость сессии

```python
# app/db/session.py
from collections.abc import AsyncGenerator
from sqlalchemy.ext.asyncio import AsyncSession
from app.db.engine import AsyncSession

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with AsyncSession() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise

# Использование в эндпоинте
@app.get("/users/{user_id}")
async def get_user(user_id: int, db: AsyncSession = Depends(get_db)):
    user = await db.get(User, user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

## Модели (Declarative)

```python
# app/models/user.py
from sqlalchemy import String, Integer, DateTime, func
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from datetime import datetime

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(Integer, primary_key=True)
    username: Mapped[str] = mapped_column(String(50), unique=True, index=True)
    email: Mapped[str] = mapped_column(String(100), unique=True)
    hashed_password: Mapped[str] = mapped_column(String(255))
    is_active: Mapped[bool] = mapped_column(default=True)
    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), server_default=func.now()
    )
    updated_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), server_default=func.now(), onupdate=func.now()
    )

    def __repr__(self):
        return f"<User {self.username}>"
```

## CRUD операции

```python
# app/services/user_service.py
from sqlalchemy import select, func
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy.orm import selectinload
from app.models.user import User

class UserService:
    def __init__(self, db: AsyncSession):
        self.db = db

    async def get_by_id(self, user_id: int) -> User | None:
        return await self.db.get(User, user_id)

    async def get_by_email(self, email: str) -> User | None:
        stmt = select(User).where(User.email == email)
        result = await self.db.execute(stmt)
        return result.scalar_one_or_none()

    async def get_all(
        self,
        skip: int = 0,
        limit: int = 10,
    ) -> tuple[list[User], int]:
        # Пагинация с общим количеством
        count_stmt = select(func.count()).select_from(User)
        total = (await self.db.execute(count_stmt)).scalar()

        stmt = select(User).order_by(User.created_at.desc()).offset(skip).limit(limit)
        result = await self.db.execute(stmt)
        users = result.scalars().all()

        return users, total

    async def create(self, username: str, email: str, password: str) -> User:
        user = User(
            username=username,
            email=email,
            hashed_password=password,  # В реальности — хэшировать!
        )
        self.db.add(user)
        await self.db.flush()  # Получаем ID без коммита
        await self.db.refresh(user)
        return user

    async def update(self, user: User, **kwargs) -> User:
        for key, value in kwargs.items():
            setattr(user, key, value)
        await self.db.flush()
        await self.db.refresh(user)
        return user

    async def delete(self, user: User) -> None:
        await self.db.delete(user)
        await self.db.flush()
```

## Relationships

```python
# app/models/post.py
from sqlalchemy import ForeignKey
from sqlalchemy.orm import relationship

class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(Integer, primary_key=True)
    title: Mapped[str] = mapped_column(String(200))
    content: Mapped[str] = mapped_column(String(5000))
    author_id: Mapped[int] = mapped_column(ForeignKey("users.id"))

    # Relationship
    author: Mapped["User"] = relationship(back_populates="posts")

# В User
class User(Base):
    # ...
    posts: Mapped[list["Post"]] = relationship(back_populates="author")
```

### Загрузка relationships

```python
# selectinload — отдельный запрос (для коллекций)
stmt = select(User).options(selectinload(User.posts)).where(User.id == 1)
result = await db.execute(stmt)
user = result.scalar_one()
print(user.posts)  # Загружено

# joinedload — JOIN (для one-to-one)
from sqlalchemy.orm import joinedload

stmt = select(Post).options(joinedload(Post.author))
```

## Alembic миграции

```bash
# Инициализация
alembic init -t async alembic

# Настройка alembic.ini
sqlalchemy.url = postgresql+asyncpg://user:pass@localhost/db

# Настройка alembic/env.py
from app.models import Base
target_metadata = Base.metadata
```

```python
# alembic/env.py (async режим)
from sqlalchemy.ext.asyncio import create_async_engine
from app.db.engine import Base

def run_migrations_online():
    connectable = create_async_engine(settings.database_url)

    async def run_async_migrations():
        async with connectable.connect() as connection:
            await connection.run_sync(do_run_migrations)
        await connectable.dispose()

    import asyncio
    asyncio.run(run_async_migrations())

def do_run_migrations(connection):
    context.configure(connection=connection, target_metadata=Base.metadata)
    with context.begin_transaction():
        context.run_migrations()
```

```bash
# Создание миграции
alembic revision --autogenerate -m "create users table"

# Применение
alembic upgrade head

# Откат
alembic downgrade -1
```

## Dependency Injection с сервисами

```python
# app/dependencies.py
from app.db.session import get_db
from app.services.user_service import UserService

async def get_user_service(db: AsyncSession = Depends(get_db)):
    return UserService(db)

# В роутере
@app.get("/users/", response_model=list[UserOut])
async def list_users(
    skip: int = 0,
    limit: int = 10,
    service: UserService = Depends(get_user_service),
):
    users, total = await service.get_all(skip=skip, limit=limit)
    return users
```

## Best Practices

✅ **Используйте** `expire_on_commit=False` для async сессий
✅ **Используйте** `get_db()` как dependency с commit/rollback
✅ **Используйте** `selectinload` для коллекций, `joinedload` для one-to-one
✅ **Используйте** сервисный слой для бизнес-логики
✅ **Используйте** `--autogenerate` для Alembic

❌ **Не используйте** синхронный SQLAlchemy с FastAPI
❌ **Не передавайте** модели БД напрямую в response
❌ **Не логируйте** пароли и чувствительные данные
❌ **Не забывайте** про `await session.rollback()` при ошибках

## Ссылки

- [FastAPI with SQLAlchemy](https://fastapi.tiangolo.com/tutorial/sql-databases/)
- [SQLAlchemy Async](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html)
- [Alembic Async](https://alembic.sqlalchemy.org/en/latest/cookbook.html#asyncio-support)

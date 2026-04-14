# PostgreSQL — Тестирование БД

## Описание

Тестирование кода работающего с БД: тестовые базы, fixtures, транзакционные тесты.

## Стратегии тестирования

| Стратегия | Скорость | Изоляция | Сложность |
|-----------|----------|----------|-----------|
| **Транзакционные тесты** | Быстро | Полная | Средняя |
| **Создание/удаление БД** | Медленно | Полная | Простая |
| **Truncate таблиц** | Средне | Полная | Средняя |
| **Мок БД** | Очень быстро | Нет | Простая |

## Транзакционные тесты (рекомендуется)

```python
# tests/conftest.py
import pytest
import asyncio
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker
from sqlalchemy.orm import DeclarativeBase

class Base(DeclarativeBase):
    pass

# Импорт всех моделей для создания таблиц
from app.models import user, post  # noqa

TEST_DATABASE_URL = "sqlite+aiosqlite:///./test.db"

@pytest.fixture(scope="session")
def anyio_backend():
    return "asyncio"

@pytest.fixture(scope="session")
def event_loop():
    loop = asyncio.new_event_loop()
    yield loop
    loop.close()

@pytest.fixture(scope="session")
async def engine():
    """Создание движка и таблиц"""
    engine = create_async_engine(TEST_DATABASE_URL, echo=False)
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield engine
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)

@pytest.fixture
async def session(engine):
    """Сессия с автооткатом после каждого теста"""
    Session = async_sessionmaker(engine, expire_on_commit=False)
    async with Session() as session:
        yield session
        await session.rollback()  # Откат всех изменений
```

### Пример теста

```python
# tests/test_users.py
import pytest
from sqlalchemy import select
from app.models.user import User

@pytest.mark.asyncio
async def test_create_user(session):
    user = User(username="alice", email="alice@example.com", hashed_password="hash")
    session.add(user)
    await session.commit()
    await session.refresh(user)

    assert user.id is not None
    assert user.username == "alice"

@pytest.mark.asyncio
async def test_get_user_by_email(session):
    # Создаём пользователя
    user = User(username="bob", email="bob@example.com", hashed_password="hash")
    session.add(user)
    await session.commit()

    # Ищем его
    stmt = select(User).where(User.email == "bob@example.com")
    result = await session.execute(stmt)
    found = result.scalar_one()

    assert found.id == user.id
    assert found.username == "bob"

# Данные не сохраняются между тестами!
@pytest.mark.asyncio
async def test_no_leftover_data(session):
    stmt = select(User)
    result = await session.execute(stmt)
    users = result.scalars().all()
    assert len(users) == 0  # Чистая БД!
```

## Создание/удаление БД (полная изоляция)

```python
# tests/conftest.py
import pytest
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker
from sqlalchemy import text

TEST_DB_NAME = "test_myapp"
ADMIN_DATABASE_URL = "postgresql+asyncpg://postgres:pass@localhost/postgres"
TEST_DATABASE_URL = f"postgresql+asyncpg://postgres:pass@localhost/{TEST_DB_NAME}"

@pytest.fixture(scope="session")
async def admin_engine():
    engine = create_async_engine(ADMIN_DATABASE_URL, isolation_level="AUTOCOMMIT")
    yield engine
    await engine.dispose()

@pytest.fixture(scope="session")
async def test_engine(admin_engine):
    """Создание тестовой БД"""
    # Создание БД
    async with admin_engine.connect() as conn:
        await conn.execute(text(f"DROP DATABASE IF EXISTS {TEST_DB_NAME}"))
        await conn.execute(text(f"CREATE DATABASE {TEST_DB_NAME}"))

    engine = create_async_engine(TEST_DATABASE_URL)
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)

    yield engine

    # Удаление БД
    async with engine.begin() as conn:
        pass
    await engine.dispose()

    async with admin_engine.connect() as conn:
        await conn.execute(text(f"DROP DATABASE IF EXISTS {TEST_DB_NAME}"))

@pytest.fixture
async def session(test_engine):
    Session = async_sessionmaker(test_engine, expire_on_commit=False)
    async with Session() as session:
        yield session
        await session.rollback()
```

## Тестирование с Alembic

```python
# tests/conftest.py
import pytest
from alembic.command import upgrade
from alembic.config import Config
from sqlalchemy.ext.asyncio import create_async_engine

@pytest.fixture(scope="session")
async def engine_with_migrations():
    engine = create_async_engine(TEST_DATABASE_URL)

    # Применение миграций
    alembic_cfg = Config("alembic.ini")
    alembic_cfg.attributes["connection"] = engine.sync_engine
    upgrade(alembic_cfg, "head")

    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)

    yield engine
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)
    await engine.dispose()
```

## Factory для тестовых данных

```python
# tests/factories.py
from app.models.user import User
from app.models.post import Post

class UserFactory:
    _counter = 0

    @classmethod
    def create(cls, **kwargs) -> User:
        cls._counter += 1
        return User(
            username=kwargs.get("username", f"user_{cls._counter}"),
            email=kwargs.get("email", f"user_{cls._counter}@example.com"),
            hashed_password=kwargs.get("hashed_password", "hash"),
            is_active=kwargs.get("is_active", True),
        )

class PostFactory:
    _counter = 0

    @classmethod
    def create(cls, author: User, **kwargs) -> Post:
        cls._counter += 1
        return Post(
            title=kwargs.get("title", f"Post {cls._counter}"),
            content=kwargs.get("content", f"Content {cls._counter}"),
            author=author,
        )

# Использование в тестах
@pytest.mark.asyncio
async def test_user_posts(session):
    user = UserFactory.create(username="alice")
    session.add(user)
    await session.commit()

    post = PostFactory.create(author=user, title="Hello")
    session.add(post)
    await session.commit()

    assert user.posts[0].title == "Hello"
```

## Тестирование сервисного слоя

```python
# tests/test_user_service.py
import pytest
from app.services.user_service import UserService
from app.models.user import User

@pytest.mark.asyncio
async def test_create_user_service(session):
    service = UserService(session)

    user = await service.create(
        username="alice",
        email="alice@example.com",
        password="secret123",
    )

    assert user.id is not None
    assert user.username == "alice"
    assert user.hashed_password != "secret123"  # Захеширован!

@pytest.mark.asyncio
async def test_get_nonexistent_user(session):
    service = UserService(session)
    user = await service.get_by_id(999)
    assert user is None

@pytest.mark.asyncio
async def test_duplicate_email(session):
    service = UserService(session)

    await service.create(username="alice", email="a@x.com", password="secret")

    with pytest.raises(Exception):  # Или ваш конкретный exception
        await service.create(username="bob", email="a@x.com", password="secret")
```

## Параметризация тестов

```python
import pytest

@pytest.mark.asyncio
@pytest.mark.parametrize("username,email,password,should_fail", [
    ("alice", "alice@example.com", "secret123", False),
    ("", "alice@example.com", "secret123", True),      # Пустой username
    ("alice", "invalid-email", "secret123", True),     # Невалидный email
    ("alice", "a@x.com", "short", True),               # Короткий пароль
])
async def test_user_validation(session, username, email, password, should_fail):
    from app.services.user_service import UserService
    service = UserService(session)

    if should_fail:
        with pytest.raises(Exception):
            await service.create(username=username, email=email, password=password)
    else:
        user = await service.create(username=username, email=email, password=password)
        assert user.id is not None
```

## Тестирование сложных запросов

```python
@pytest.mark.asyncio
async def test_pagination(session):
    from app.services.post_service import PostService
    service = PostService(session)

    # Создаём 25 постов
    user = UserFactory.create()
    session.add(user)
    await session.commit()

    for _ in range(25):
        post = PostFactory.create(author=user)
        session.add(post)
    await session.commit()

    # Пагинация
    posts, total = await service.get_all(skip=0, limit=10)
    assert len(posts) == 10
    assert total == 25

    posts, total = await service.get_all(skip=10, limit=10)
    assert len(posts) == 10

    posts, total = await service.get_all(skip=20, limit=10)
    assert len(posts) == 5
```

## Best Practices

✅ **Используйте** транзакционные тесты (rollback после каждого теста)
✅ **Используйте** factory для создания тестовых данных
✅ **Тестируйте** сервисный слой, а не только роутеры
✅ **Параметризуйте** тесты для edge cases
✅ **Используйте** `scope="session"` для создания таблиц

❌ **Не используйте** продакшен БД для тестов
❌ **Не оставляйте** данные между тестами
❌ **Не тестируйте** SQL напрямую — тестируйте сервисный слой
❌ **Не используйте** `VACUUM FULL` или `DROP TABLE` в тестах

## Ссылки

- [pytest-asyncio](https://pytest-asyncio.readthedocs.io/)
- [SQLAlchemy Testing](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html#preventing-io-in-tests)

# FastAPI — Продвинутое тестирование

## Описание

Тестирование FastAPI приложений: TestClient, моки зависимостей, fixtures, async тесты.

## Установка

```bash
pip install pytest pytest-asyncio httpx
```

## Базовое тестирование

```python
# tests/conftest.py
import pytest
from fastapi.testclient import TestClient
from app.main import create_app

@pytest.fixture
def client():
    """Создание тестового клиента"""
    app = create_app()
    with TestClient(app) as c:
        yield c

# tests/test_users.py
def test_read_users(client: TestClient):
    response = client.get("/api/v1/users/")
    assert response.status_code == 200
    assert isinstance(response.json(), list)

def test_create_user(client: TestClient):
    response = client.post(
        "/api/v1/users/",
        json={
            "username": "alice",
            "email": "alice@example.com",
            "password": "secret123",
        },
    )
    assert response.status_code == 201
    data = response.json()
    assert data["username"] == "alice"
    assert "password" not in data  # Не должно быть в ответе
```

## Моки зависимостей

```python
# tests/conftest.py
from unittest.mock import AsyncMock, patch
from app.dependencies import get_db, get_user_service, get_current_user
from app.models.user import User

@pytest.fixture
def mock_db_session():
    """Мок сессии БД"""
    session = AsyncMock()
    session.get = AsyncMock(return_value=None)
    session.execute = AsyncMock()
    session.add = Mock()
    session.flush = AsyncMock()
    session.refresh = AsyncMock()
    session.commit = AsyncMock()
    session.rollback = AsyncMock()
    return session

@pytest.fixture
def mock_user_service():
    """Мок UserService"""
    service = AsyncMock()
    service.get_by_id.return_value = None
    service.get_by_email.return_value = None
    service.create.return_value = User(
        id=1, username="alice", email="alice@example.com", is_active=True
    )
    return service

@pytest.fixture
def mock_current_user():
    """Мок текущего пользователя"""
    return User(id=1, username="alice", email="alice@example.com", is_active=True)

# Использование моков
@pytest.fixture
def client_with_mocks(mock_db_session, mock_user_service, mock_current_user):
    app = create_app()

    # Переопределение зависимостей
    async def override_get_db():
        yield mock_db_session

    async def override_get_user_service():
        return mock_user_service

    async def override_get_current_user():
        return mock_current_user

    app.dependency_overrides[get_db] = override_get_db
    app.dependency_overrides[get_user_service] = override_get_user_service
    app.dependency_overrides[get_current_user] = override_get_current_user

    with TestClient(app) as c:
        yield c

    # Очистка
    app.dependency_overrides.clear()

def test_get_user_with_mock(client_with_mocks, mock_user_service):
    response = client_with_mocks.get("/api/v1/users/1")
    assert response.status_code == 200
    mock_user_service.get_by_id.assert_called_once_with(1)
```

## Тестирование с реальной БД

```python
# tests/conftest.py
import pytest
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker
from app.models.base import Base

@pytest.fixture(scope="session")
def anyio_backend():
    return "asyncio"

@pytest.fixture
async def db_engine():
    """Создание тестовой БД"""
    engine = create_async_engine("sqlite+aiosqlite:///./test.db")
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield engine
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)

@pytest.fixture
async def db_session(db_engine):
    """Сессия с автооткатом"""
    Session = async_sessionmaker(db_engine)
    async with Session() as session:
        yield session
        await session.rollback()

@pytest.fixture
async def client_with_db(db_engine, db_session):
    app = create_app()

    async def override_get_db():
        yield db_session

    app.dependency_overrides[get_db] = override_get_db

    from httpx import AsyncClient
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac

    app.dependency_overrides.clear()

@pytest.mark.asyncio
async def test_create_user_real_db(client_with_db):
    response = await client_with_db.post(
        "/api/v1/users/",
        json={
            "username": "bob",
            "email": "bob@example.com",
            "password": "secret123",
        },
    )
    assert response.status_code == 201
    data = response.json()
    assert data["username"] == "bob"

    # Проверка что пользователь в БД
    response = await client_with_db.get(f"/api/v1/users/{data['id']}")
    assert response.status_code == 200
```

## Тестирование аутентификации

```python
def test_protected_endpoint_without_auth(client):
    response = client.get("/api/v1/users/me")
    assert response.status_code == 401

def test_protected_endpoint_with_auth(client, mock_current_user):
    # Получение токена
    token_response = client.post(
        "/auth/token",
        data={"username": "alice", "password": "secret"},
    )
    token = token_response.json()["access_token"]

    # Запрос с токеном
    response = client.get(
        "/api/v1/users/me",
        headers={"Authorization": f"Bearer {token}"},
    )
    assert response.status_code == 200

# Моки аутентификации
@pytest.fixture
def authenticated_client(client):
    """Клиент с замоканной аутентификацией"""
    from app.dependencies import get_current_user
    from app.models.user import User

    fake_user = User(id=1, username="alice", email="a@x.com", is_active=True)

    async def override_get_current_user():
        return fake_user

    client.app.dependency_overrides[get_current_user] = override_get_current_user
    yield client
    client.app.dependency_overrides.clear()

def test_auth_with_mock(authenticated_client):
    response = authenticated_client.get("/api/v1/users/me")
    assert response.status_code == 200
    assert response.json()["username"] == "alice"
```

## Параметризация

```python
import pytest

@pytest.mark.parametrize(
    "username,email,password,expected_status",
    [
        ("alice", "alice@example.com", "secret123", 201),
        ("", "alice@example.com", "secret123", 422),  # Пустой username
        ("alice", "invalid-email", "secret123", 422),  # Невалидный email
        ("alice", "alice@example.com", "short", 422),  # Короткий пароль
        ("bob", "alice@example.com", "secret123", 400),  # Email занят
    ],
)
def test_create_user_validation(
    client, username, email, password, expected_status
):
    response = client.post(
        "/api/v1/users/",
        json={
            "username": username,
            "email": email,
            "password": password,
        },
    )
    assert response.status_code == expected_status
```

## Тестирование middleware

```python
def test_cors_headers(client):
    response = client.options(
        "/api/v1/users/",
        headers={"Origin": "http://localhost:3000"},
    )
    assert response.headers["access-control-allow-origin"] == "http://localhost:3000"

def test_request_logging(caplog):
    with caplog.at_level("INFO"):
        client.get("/api/v1/users/")
        assert "GET /api/v1/users/" in caplog.text
```

## Best Practices

✅ **Используйте** `dependency_overrides` для моков
✅ **Используйте** `pytest.fixture` для переиспользуемых компонентов
✅ **Тестируйте** валидацию входных данных
✅ **Тестируйте** ошибки и edge cases
✅ **Используйте** `parameterize` для множественных сценариев
✅ **Очищайте** `dependency_overrides` после тестов

❌ **Не используйте** реальные API в unit тестах
❌ **Не тестируйте** сторонние библиотеки
❌ **Не забывайте** очищать mock после тестов

## Ссылки

- [FastAPI Testing](https://fastapi.tiangolo.com/tutorial/testing/)
- [TestClient документация](https://www.starlette.io/testclient/)

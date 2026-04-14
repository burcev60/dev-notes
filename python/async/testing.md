# asyncio — Тестирование (pytest-asyncio)

## Описание

Тестирование асинхронного кода с помощью `pytest-asyncio`.

## Установка

```bash
pip install pytest pytest-asyncio
```

## Базовые тесты

```python
import pytest
import asyncio

# Режимы: auto, strict, manual
# pytest.ini: asyncio_mode = auto

@pytest.mark.asyncio
async def test_async_function():
    result = await async_greet("Alice")
    assert result == "Hello, Alice!"

@pytest.mark.asyncio
async def test_async_with_fixture():
    async with async_resource() as resource:
        assert resource.status == "open"
```

## Фикстуры

```python
import pytest

@pytest.fixture
def anyio_backend():
    return "asyncio"

@pytest.mark.asyncio
async def test_with_fixture(my_fixture):
    result = await my_fixture.do_something()
    assert result is not None

# Async фикстуры
@pytest.fixture
async def db_session():
    """Создание и очистка БД"""
    session = await create_test_session()
    yield session
    await session.close()
    await drop_test_tables()

@pytest.mark.asyncio
async def test_database(db_session):
    user = User(name="Alice")
    db_session.add(user)
    await db_session.commit()

    result = await db_session.execute(select(User))
    users = result.scalars().all()
    assert len(users) == 1
```

## Конфигурация pytest

```ini
# pytest.ini
[pytest]
asyncio_mode = auto
asyncio_default_fixture_loop_scope = function
```

```python
# pyproject.toml
[tool.pytest.ini_options]
asyncio_mode = "auto"
asyncio_default_fixture_loop_scope = "function"
```

### Режимы asyncio_mode

| Режим | Описание |
|-------|----------|
| `auto` | Все async тесты автоматически запускаются |
| `strict` | Нужен `@pytest.mark.asyncio` |
| `manual` | Полный контроль (нужен event_loop fixture) |

## Тестирование с моками

```python
from unittest.mock import AsyncMock, patch, MagicMock

@pytest.mark.asyncio
async def test_with_async_mock():
    # Async мок
    mock_client = AsyncMock()
    mock_client.get.return_value = {"id": 1, "name": "Alice"}

    result = await mock_client.get("/users/1")
    assert result == {"id": 1, "name": "Alice"}
    mock_client.get.assert_awaited_once_with("/users/1")

# Патчинг async функций
@pytest.mark.asyncio
@patch("mymodule.fetch_data", new_callable=AsyncMock)
async def test_with_patch(mock_fetch):
    mock_fetch.return_value = {"data": "mocked"}

    result = await my_function()
    assert result["data"] == "mocked"
    mock_fetch.assert_awaited_once()
```

## Тестирование с timeout

```python
import pytest

@pytest.mark.asyncio
async def test_timeout():
    with pytest.raises(asyncio.TimeoutError):
        await asyncio.wait_for(slow_operation(), timeout=0.1)

@pytest.mark.asyncio
async def test_completes_in_time():
    result = await asyncio.wait_for(fast_operation(), timeout=5.0)
    assert result is not None
```

## Тестирование задач

```python
@pytest.mark.asyncio
async def test_task_creation():
    async def background_work():
        await asyncio.sleep(0.1)
        return "done"

    task = asyncio.create_task(background_work())
    result = await task
    assert result == "done"

@pytest.mark.asyncio
async def test_task_cancellation():
    async def long_running():
        try:
            await asyncio.sleep(10)
        except asyncio.CancelledError:
            raise

    task = asyncio.create_task(long_running())
    await asyncio.sleep(0.1)
    task.cancel()

    with pytest.raises(asyncio.CancelledError):
        await task
```

## Тестирование Queue

```python
@pytest.mark.asyncio
async def test_queue():
    queue = asyncio.Queue()

    await queue.put("item1")
    await queue.put("item2")

    assert queue.qsize() == 2
    assert await queue.get() == "item1"
    assert await queue.get() == "item2"
    assert queue.empty()

@pytest.mark.asyncio
async def test_producer_consumer():
    queue = asyncio.Queue()
    results = []

    async def producer():
        for i in range(3):
            await queue.put(i)

    async def consumer():
        while len(results) < 3:
            item = await queue.get()
            results.append(item)
            queue.task_done()

    await asyncio.gather(producer(), consumer())
    assert results == [0, 1, 2]
```

## Тестирование веб-приложений

### FastAPI

```python
import pytest
from fastapi.testclient import TestClient
from httpx import AsyncClient

# Синхронное тестирование
def test_fastapi_sync():
    client = TestClient(app)
    response = client.get("/users/")
    assert response.status_code == 200

# Асинхронное тестирование
@pytest.mark.asyncio
async def test_fastapi_async():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        response = await ac.get("/users/")
        assert response.status_code == 200

@pytest.mark.asyncio
async def test_fastapi_post():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        response = await ac.post(
            "/users/",
            json={"name": "Alice", "email": "alice@example.com"},
        )
        assert response.status_code == 201
        assert response.json()["name"] == "Alice"
```

### aiohttp

```python
import pytest
from aiohttp import web
from aiohttp.test_utils import AioHTTPTestCase, unittest_run_loop

class TestMyApp(AioHTTPTestCase):
    async def get_application(self):
        return create_app()

    async def test_get_users(self):
        resp = await self.client.get("/users/")
        assert resp.status == 200
        data = await resp.json()
        assert len(data) == 0
```

## Параметризация

```python
import pytest

@pytest.mark.asyncio
@pytest.mark.parametrize("input_val,expected", [
    (1, 1),
    (2, 4),
    (3, 9),
    (-1, 1),
])
async def test_square(input_val: int, expected: int):
    result = await async_square(input_val)
    assert result == expected

@pytest.mark.asyncio
@pytest.mark.parametrize("url,status_code", [
    ("/users", 200),
    ("/users/1", 200),
    ("/nonexistent", 404),
])
async def test_endpoints(url: str, status_code: int):
    async with AsyncClient(app=app, base_url="http://test") as ac:
        response = await ac.get(url)
        assert response.status_code == status_code
```

## Фикстуры с областью видимости

```python
import pytest

@pytest.fixture(scope="session")
def event_loop():
    """Один event loop на все тесты"""
    loop = asyncio.new_event_loop()
    yield loop
    loop.close()

@pytest.fixture(scope="session")
async def db_engine():
    """Создание БД один раз на все тесты"""
    engine = create_async_engine("postgresql+asyncpg://localhost/test_db")
    await create_tables(engine)
    yield engine
    await engine.dispose()

@pytest.fixture
async def db_session(db_engine):
    """Новая сессия для каждого теста"""
    async with db_engine.begin() as conn:
        await conn.begin()
    yield conn
    await conn.rollback()
```

## Тестирование стриминга

```python
@pytest.mark.asyncio
async def test_streaming_response():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        async with ac.stream("GET", "/events") as response:
            assert response.status_code == 200
            chunks = []
            async for chunk in response.aiter_text():
                chunks.append(chunk)
            assert len(chunks) > 0
```

## Best Practices

✅ **Используйте** `asyncio_mode = auto` в pytest.ini
✅ **Используйте** `AsyncMock` для моков async функций
✅ **Используйте** `assert_awaited_once()` для проверки вызовов
✅ **Используйте** `AsyncClient` для FastAPI тестов
✅ **Разделяйте** фикстуры по scope (session/function)

❌ **Не используйте** `time.sleep()` в тестах — используйте `asyncio.sleep()`
❌ **Не забывайте** закрывать соединения в teardown
❌ **Не тестируйте** асинхронный код синхронными тестами
❌ **Не создавайте** новый event loop для каждого теста без причины

## Ссылки

- [pytest-asyncio документация](https://pytest-asyncio.readthedocs.io/)
- [FastAPI тестирование](https://fastapi.tiangolo.com/tutorial/testing/)
- [unittest.mock AsyncMock](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.AsyncMock)

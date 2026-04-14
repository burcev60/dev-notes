# FastAPI — Тестирование

## Описание

FastAPI предоставляет `TestClient` для тестирования без запуска сервера.

## Базовое тестирование

```python
from fastapi.testclient import TestClient
from .main import app

client = TestClient(app)

def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello World"}

def test_create_item():
    response = client.post("/items/", json={
        "name": "Test",
        "price": 10.0
    })
    assert response.status_code == 201
    assert response.json()["name"] == "Test"
```

## Тестирование с моками

```python
def test_get_user_with_mock():
    def mock_get_db():
        return MockDB()

    app.dependency_overrides[get_db] = mock_get_db

    response = client.get("/users/1")
    assert response.status_code == 200

    app.dependency_overrides.clear()
```

## Async тестирование

```python
import pytest
from httpx import AsyncClient

@pytest.mark.anyio
async def test_async_endpoint():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        response = await ac.get("/")
    assert response.status_code == 200
```

## Best Practices

✅ **Используйте** `TestClient` для интеграционных тестов
✅ **Используйте** `dependency_overrides` для моков

## Ссылки

- [FastAPI Testing](https://fastapi.tiangolo.com/tutorial/testing/)

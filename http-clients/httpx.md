# httpx — Асинхронный HTTP клиент

## Описание

`httpx` — современный HTTP клиент с поддержкой как синхронного, так и асинхронного режима.

## Установка

```bash
pip install httpx
```

## Синхронное использование

```python
import httpx

response = httpx.get("https://api.example.com/users")
print(response.status_code, response.json())

response = httpx.post(
    "https://api.example.com/users",
    json={"name": "Alice"}
)
```

## Асинхронное использование

```python
import httpx

async with httpx.AsyncClient() as client:
    response = await client.get("https://api.example.com/users")
    print(response.json())

    response = await client.post(
        "https://api.example.com/users",
        json={"name": "Bob"}
    )
```

## Клиент с конфигурацией

```python
client = httpx.Client(
    base_url="https://api.example.com",
    timeout=10.0,
    headers={"Authorization": "Bearer token"}
)

response = client.get("/users")  # https://api.example.com/users
```

## Best Practices

✅ **Используйте** `AsyncClient` для FastAPI
✅ **Используйте** `base_url` для API клиентов
✅ **Настраивайте** `timeout`

## Ссылки

- [httpx документация](https://www.python-httpx.org/)

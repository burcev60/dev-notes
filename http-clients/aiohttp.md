# aiohttp — Асинхронный HTTP клиент/сервер

## Описание

`aiohttp` — асинхронный HTTP клиент и серверный фреймворк.

## Установка

```bash
pip install aiohttp
```

## Клиент

```python
import aiohttp

async with aiohttp.ClientSession() as session:
    async with session.get("https://api.example.com/users") as response:
        data = await response.json()
        print(data)

    async with session.post(
        "https://api.example.com/users",
        json={"name": "Alice"}
    ) as response:
        result = await response.json()
```

## Параллельные запросы

```python
async with aiohttp.ClientSession() as session:
    tasks = [
        session.get(f"https://api.example.com/users/{i}")
        for i in range(10)
    ]
    responses = await asyncio.gather(*[t for t in tasks])
```

## Best Practices

✅ **Используйте** `ClientSession` для множества запросов
✅ **Закрывайте** сессию через `async with`

## Ссылки

- [aiohttp документация](https://docs.aiohttp.org/)

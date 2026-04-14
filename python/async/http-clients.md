# asyncio — HTTP запросы (aiohttp, httpx)

## Описание

Асинхронные HTTP-клиенты для выполнения неблокирующих запросов.

## httpx.AsyncClient — Современный async HTTP

```python
import httpx

# Контекстный менеджер (рекомендуется)
async with httpx.AsyncClient() as client:
    response = await client.get("https://api.example.com/users")
    print(response.status_code, response.json())

# Без контекста (нужно закрывать вручную)
client = httpx.AsyncClient()
try:
    response = await client.get("https://api.example.com/users")
finally:
    await client.aclose()
```

### Конфигурация клиента

```python
async with httpx.AsyncClient(
    base_url="https://api.example.com",
    timeout=httpx.Timeout(10.0, connect=5.0),
    limits=httpx.Limits(max_connections=100, max_keepalive_connections=20),
    headers={"Authorization": "Bearer token"},
    follow_redirects=True,
    http2=True,  # HTTP/2 поддержка
) as client:
    response = await client.get("/users")
```

### Параллельные запросы

```python
async def fetch_multiple_urls():
    urls = [
        "https://api.example.com/users",
        "https://api.example.com/posts",
        "https://api.example.com/comments",
    ]

    async with httpx.AsyncClient() as client:
        tasks = [client.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)

        for response in responses:
            print(f"{response.url}: {response.status_code}")

# С обработкой ошибок
async def fetch_with_errors():
    urls = ["https://api.example.com/ok", "https://bad-url.invalid"]

    async with httpx.AsyncClient() as client:
        tasks = [client.get(url, follow_redirects=True) for url in urls]
        results = await asyncio.gather(*tasks, return_exceptions=True)

        for result in results:
            if isinstance(result, Exception):
                print(f"Error: {result}")
            else:
                print(f"{result.status_code}: {result.url}")
```

### Streaming

```python
async def stream_download():
    async with httpx.AsyncClient() as client:
        async with client.stream("GET", "https://example.com/large-file") as response:
            response.raise_for_status()
            with open("downloaded.bin", "wb") as f:
                async for chunk in response.aiter_bytes(chunk_size=8192):
                    f.write(chunk)

# Streaming JSON (line-delimited)
async def stream_events():
    async with httpx.AsyncClient() as client:
        async with client.stream("GET", "https://api.example.com/events") as response:
            async for line in response.aiter_lines():
                event = json.loads(line)
                print(f"Event: {event}")
```

### Механизм middleware

```python
class LoggingTransport(httpx.AsyncBaseTransport):
    def __init__(self, transport: httpx.AsyncBaseTransport):
        self._transport = transport

    async def handle_async_request(self, request: httpx.Request) -> httpx.Response:
        start = time.time()
        response = await self._transport.handle_async_request(request)
        duration = time.time() - start
        print(f"{request.method} {request.url} - {response.status_code} ({duration:.3f}s)")
        return response

# Использование
transport = httpx.AsyncHTTPTransport()
logging_transport = LoggingTransport(transport)

async with httpx.AsyncClient(transport=logging_transport) as client:
    await client.get("https://api.example.com")
```

## aiohttp — Async HTTP клиент и сервер

### Клиент

```python
import aiohttp

async with aiohttp.ClientSession() as session:
    async with session.get("https://api.example.com/users") as response:
        data = await response.json()
        print(data)

    async with session.post(
        "https://api.example.com/users",
        json={"name": "Alice"},
    ) as response:
        result = await response.json()
```

### Конфигурация ClientSession

```python
timeout = aiohttp.ClientTimeout(total=30, connect=10, sock_read=20)
connector = aiohttp.TCPConnector(
    limit=100,           # Общее ограничение соединений
    limit_per_host=10,   # На один хост
    ttl_dns_cache=300,   # DNS кэш
)

async with aiohttp.ClientSession(
    base_url="https://api.example.com",
    timeout=timeout,
    connector=connector,
    headers={"Authorization": "Bearer token"},
) as session:
    response = await session.get("/users")
```

### Параллельные запросы

```python
async def fetch_all(session: aiohttp.ClientSession, urls: list[str]) -> list[dict]:
    async def fetch(url: str):
        async with session.get(url) as response:
            return await response.json()

    tasks = [fetch(url) for url in urls]
    return await asyncio.gather(*tasks)

# Semaphore для ограничения
async def fetch_with_limit(session: aiohttp.ClientSession, urls: list[str]):
    semaphore = asyncio.Semaphore(10)

    async def fetch(url: str):
        async with semaphore:
            async with session.get(url) as response:
                return await response.json()

    tasks = [fetch(url) for url in urls]
    return await asyncio.gather(*tasks)
```

### Retry middleware

```python
import aiohttp
from aiohttp_retry import RetryClient, ExponentialRetry

async def fetch_with_retry():
    retry_options = ExponentialRetry(
        attempts=3,
        start_timeout=0.1,
        statuses={500, 502, 503, 504},
    )

    async with RetryClient(raise_for_status=True, retry_options=retry_options) as client:
        async with client.get("https://api.example.com/unstable") as response:
            data = await response.json()
```

### Сервер (aiohttp)

```python
from aiohttp import web

async def handle_get(request: web.Request) -> web.Response:
    user_id = request.match_info["id"]
    return web.json_response({"user_id": user_id})

async def handle_post(request: web.Request) -> web.Response:
    data = await request.json()
    return web.json_response({"created": data}, status=201)

app = web.Application()
app.router.add_get("/users/{id}", handle_get)
app.router.add_post("/users", handle_post)

if __name__ == "__main__":
    web.run_app(app, host="0.0.0.0", port=8080)
```

## Сравнение httpx vs aiohttp

| Фича | httpx | aiohttp |
|------|-------|---------|
| Синхронный режим | ✅ | ❌ |
| HTTP/2 | ✅ | ❌ |
| Сервер | ❌ | ✅ |
| WebSockets | ✅ (через wsproto) | ✅ |
| Streaming | ✅ | ✅ |
| Middleware | Транспорт | Декораторы |
| Активность | Активен | Активен |

## Best Practices

✅ **Используйте** `async with` для автоматического закрытия
✅ **Используйте** `base_url` для API клиентов
✅ **Настраивайте** `timeout` и `limits`
✅ **Используйте** `return_exceptions=True` с `gather`
✅ **Используйте** `Semaphore` для rate limiting

❌ **Не создавайте** новую сессию на каждый запрос
❌ **Не забывайте** `timeout` — запросы могут зависнуть
❌ **Не используйте** `aiohttp` без `ClientSession` (антипаттерн)
❌ **Не игнорируйте** `response.raise_for_status()` для критичных запросов

## Ссылки

- [httpx документация](https://www.python-httpx.org/)
- [aiohttp документация](https://docs.aiohttp.org/)
- [aiohttp_retry](https://github.com/inyutin/aiohttp_retry)

# FastAPI — Кэширование

## Описание

Кэширование ответов для повышения производительности: in-memory, Redis, fastapi-cache.

## In-memory кэш (functools.lru_cache)

```python
from functools import lru_cache
from fastapi import FastAPI

app = FastAPI()

@lru_cache(maxsize=128)
def get_config():
    """Кэширование конфигурации"""
    # Загрузка из файла/БД
    return {"api_key": "...", "timeout": 30}

@app.get("/config")
async def config():
    return get_config()
```

### Асинхронный кэш

```python
from functools import lru_cache
import asyncio

_cache = {}

async def async_cache(key: str, ttl: int = 60):
    """Простой async кэш с TTL"""
    if key in _cache:
        value, expires_at = _cache[key]
        if asyncio.get_event_loop().time() < expires_at:
            return value
        del _cache[key]
    return None

async def set_cache(key: str, value: Any, ttl: int = 60):
    loop = asyncio.get_event_loop()
    _cache[key] = (value, loop.time() + ttl)
```

## Redis кэш

```python
# pip install redis
import redis.asyncio as redis
import json
from fastapi import FastAPI

app = FastAPI()
redis_client = redis.Redis(host="localhost", port=6379, db=0, decode_responses=True)

@app.get("/items/{item_id}")
async def get_item(item_id: int):
    # Проверка кэша
    cache_key = f"item:{item_id}"
    cached = await redis_client.get(cache_key)

    if cached:
        return json.loads(cached)

    # Запрос к БД
    item = await db.get(Item, item_id)
    if not item:
        raise HTTPException(status_code=404)

    # Сохранение в кэш (10 минут)
    await redis_client.setex(
        cache_key,
        600,
        json.dumps({"id": item.id, "name": item.name}),
    )

    return item

@app.delete("/items/{item_id}")
async def delete_item(item_id: int):
    # Инвалидация кэша
    await redis_client.delete(f"item:{item_id}")
    # Удаление из БД
    ...
```

## fastapi-cache

```python
# pip install fastapi-cache2[redis]
from fastapi_cache import FastAPICache
from fastapi_cache.backends.redis import RedisBackend
from fastapi_cache.decorator import cache
import redis.asyncio as redis

# Инициализация
@app.on_event("startup")
async def startup():
    redis_client = redis.Redis(host="localhost", port=6379, db=0)
    FastAPICache.init(RedisBackend(redis_client), prefix="fastapi-cache")

# Кэширование эндпоинта
@cache(expire=60)  # 60 секунд
@app.get("/items/{item_id}")
async def get_item(item_id: int):
    return await db.get(Item, item_id)

# С кастомным ключом
@cache(expire=300, coder=JsonCoder())
@app.get("/users/{user_id}/posts")
async def get_user_posts(user_id: int, skip: int = 0, limit: int = 10):
    return await post_service.get_by_user(user_id, skip, limit)
```

### Инвалидация fastapi-cache

```python
from fastapi_cache import FastAPICache

async def invalidate_user_cache(user_id: int):
    backend = FastAPICache.get_backend()
    await backend.clear(f"users:{user_id}")
    await backend.clear(f"users:{user_id}:posts")
```

## Кэш с декоратором

```python
from functools import wraps
import json
import redis.asyncio as redis

redis_client = redis.Redis(host="localhost", port=6379, db=0)

def cache_response(ttl: int = 60, prefix: str = ""):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # Генерация ключи
            key = f"{prefix}:{func.__name__}:{args}:{kwargs}"

            # Проверка кэша
            cached = await redis_client.get(key)
            if cached:
                return json.loads(cached)

            # Вызов функции
            result = await func(*args, **kwargs)

            # Сохранение
            await redis_client.setex(key, ttl, json.dumps(result, default=str))
            return result
        return wrapper
    return decorator

@cache_response(ttl=300, prefix="users")
async def get_all_users():
    return [{"id": 1, "name": "Alice"}]
```

## HTTP Cache-Control

```python
from fastapi import FastAPI, Response
from fastapi.responses import JSONResponse

app = FastAPI()

@app.get("/items/{item_id}")
async def get_item(item_id: int, response: Response):
    item = await db.get(Item, item_id)

    # HTTP кэширование
    response.headers["Cache-Control"] = "public, max-age=300"  # 5 минут
    response.headers["ETag"] = f'"{item.updated_at.timestamp()}"'

    return item

# Conditional requests
from fastapi import Request

@app.get("/items/{item_id}")
async def get_item_conditional(item_id: int, request: Request, response: Response):
    item = await db.get(Item, item_id)
    etag = f'"{item.updated_at.timestamp()}"'

    # Проверка If-None-Match
    if request.headers.get("if-none-match") == etag:
        return Response(status_code=304)

    response.headers["ETag"] = etag
    response.headers["Cache-Control"] = "public, max-age=300"
    return item
```

## Best Practices

✅ **Используйте** Redis для распределённого кэша
✅ **Используйте** `lru_cache` для конфигурации и констант
✅ **Инвалидируйте** кэш при изменении данных
✅ **Устанавливайте** разумный TTL (зависит от данных)
✅ **Используйте** HTTP Cache-Control для статики

❌ **Не кэшируйте** персональные данные без проверки авторизации
❌ **Не забывайте** про инвалидацию
❌ **Не кэшируйте** ответы с ошибками
❌ **Не используйте** in-memory кэш для масштабируемых приложений

## Ссылки

- [fastapi-cache2](https://github.com/long2ice/fastapi-cache)
- [Redis документация](https://redis.io/docs/)
- [HTTP Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)

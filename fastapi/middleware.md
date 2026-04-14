# FastAPI — Middleware

## Описание

Middleware обрабатывают запросы/ответы до и после view функций.

## Встроенный middleware

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)
```

## Custom middleware

```python
from starlette.middleware.base import BaseHTTPMiddleware

class LoggingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        start = time.time()
        response = await call_next(request)
        duration = time.time() - start
        print(f"{request.method} {request.url} - {duration:.3f}s")
        return response

app.add_middleware(LoggingMiddleware)
```

## Best Practices

✅ **Используйте** middleware для логирования, CORS, rate limiting

## Ссылки

- [FastAPI Middleware](https://fastapi.tiangolo.com/tutorial/middleware/)

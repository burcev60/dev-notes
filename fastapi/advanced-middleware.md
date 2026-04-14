# FastAPI — Middleware: CORS, Rate limiting, GZip, Tracing

## Описание

Продвинутые middleware для безопасности, производительности и мониторинга.

## CORS (Cross-Origin Resource Sharing)

```python
from fastapi.middleware.cors import CORSMiddleware

# Базовая настройка
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://example.com"],           # Разрешённые origins
    allow_credentials=True,                          # Разрешить cookies
    allow_methods=["GET", "POST", "PUT", "DELETE"],  # Методы
    allow_headers=["*"],                             # Заголовки
    max_age=600,                                     # Кэш preflight (секунды)
)

# Все origins (только для dev!)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

# С настройками из config
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.cors_origins,
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "PATCH", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
    max_age=3600,
)
```

## Rate Limiting

### Простой rate limiter

```python
from fastapi import Request, HTTPException
from starlette.middleware.base import BaseHTTPMiddleware
import time
from collections import defaultdict

class RateLimitMiddleware(BaseHTTPMiddleware):
    def __init__(self, app, max_requests: int = 100, window: int = 60):
        super().__init__(app)
        self.max_requests = max_requests
        self.window = window
        self.requests: dict[str, list[float]] = defaultdict(list)

    async def dispatch(self, request: Request, call_next):
        client_ip = request.client.host if request.client else "unknown"
        now = time.time()

        # Очистка старых запросов
        self.requests[client_ip] = [
            t for t in self.requests[client_ip] if now - t < self.window
        ]

        # Проверка лимита
        if len(self.requests[client_ip]) >= self.max_requests:
            raise HTTPException(
                status_code=429,
                detail="Rate limit exceeded",
                headers={"Retry-After": str(self.window)},
            )

        # Добавление запроса
        self.requests[client_ip].append(now)

        response = await call_next(request)
        response.headers["X-RateLimit-Limit"] = str(self.max_requests)
        response.headers["X-RateLimit-Remaining"] = str(
            self.max_requests - len(self.requests[client_ip])
        )
        return response

app.add_middleware(RateLimitMiddleware, max_requests=100, window=60)
```

### Rate limiting через Redis

```python
# pip install slowapi
from slowapi import Limiter
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

limiter = Limiter(key_func=get_remote_address)
app = FastAPI()
app.state.limiter = limiter

@app.exception_handler(RateLimitExceeded)
async def rate_limit_handler(request: Request, exc: RateLimitExceeded):
    return JSONResponse(
        status_code=429,
        content={"detail": "Rate limit exceeded"},
    )

# На эндпоинте
@app.get("/items/")
@limiter.limit("10/minute")
async def get_items(request: Request):
    return {"items": []}

# Для всех эндпоинтов
app.add_middleware(
    RateLimitMiddleware,
    rate_limit="100/minute",
)
```

## GZip сжатие

```python
from fastapi.middleware.gzip import GZipMiddleware

# Сжатие ответов > 1KB
app.add_middleware(GZipMiddleware, minimum_size=1000, compresslevel=6)

# compresslevel: 1 (быстро) → 9 (максимальное сжатие)
```

## HTTPS Redirect

```python
from starlette.middleware.httpsredirect import HTTPSRedirectMiddleware

# Перенаправление HTTP → HTTPS
app.add_middleware(HTTPSRedirectMiddleware)
```

## Trusted Host

```python
from starlette.middleware.trustedhost import TrustedHostMiddleware

# Разрешение только определённых хостов
app.add_middleware(
    TrustedHostMiddleware,
    allowed_hosts=["example.com", "*.example.com"],
)
```

## Request Tracing (OpenTelemetry)

```python
# pip install opentelemetry-api opentelemetry-sdk opentelemetry-instrumentation-fastapi
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor

trace.set_tracer_provider(TracerProvider())

app = FastAPI()
FastAPIInstrumentor.instrument_app(app)
```

## Custom middleware с трассировкой

```python
import time
import uuid
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware

class TracingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        # Генерация request ID
        request_id = str(uuid.uuid4())
        request.state.request_id = request_id

        start = time.time()

        response = await call_next(request)

        duration = time.time() - start

        # Добавление заголовков
        response.headers["X-Request-ID"] = request_id
        response.headers["X-Response-Time"] = f"{duration:.3f}s"

        return response

app.add_middleware(TracingMiddleware)
```

## Порядок middleware

```python
# ВАЖНО: middleware выполняются в обратном порядке добавления!

# 1. Добавлен последним — выполняется первым
app.add_middleware(TracingMiddleware)           # 1. Внешний

# 2. 
app.add_middleware(RateLimitMiddleware)          # 2.

# 3. 
app.add_middleware(GZipMiddleware, minimum_size=1000)  # 3.

# 4. Добавлен первым — выполняется последним
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.cors_origins,
)                                               # 4. Внутренний
```

## Middleware vs Dependency

| Характеристика | Middleware | Dependency |
|---------------|-----------|------------|
| **Уровень** | Приложение | Эндпоинт/Роутер |
| **Доступ** | Все запросы | Конкретные эндпоинты |
| **Scope** | Global | Локальный |
| **Примеры** | CORS, GZip, Logging | Аутентификация, БД |

## Best Practices

✅ **Настраивайте** CORS только для нужных origins
✅ **Используйте** rate limiting для публичных API
✅ **Включайте** GZip для больших ответов
✅ **Добавляйте** request ID для трассировки
✅ **Используйте** OpenTelemetry для distributed tracing

❌ **Не используйте** `allow_origins=["*"]` в продакшене
❌ **Не ставьте** слишком высокий rate limit
❌ **Не забывайте** про порядок middleware
❌ **Не логируйте** sensitive данные в middleware

## Ссылки

- [FastAPI Middleware](https://fastapi.tiangolo.com/tutorial/middleware/)
- [CORS документация](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [SlowAPI](https://github.com/laurentS/slowapi)
- [OpenTelemetry](https://opentelemetry.io/)

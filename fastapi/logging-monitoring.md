# FastAPI — Логирование и мониторинг

## Описание

Логирование запросов, мониторинг производительности, интеграция с Prometheus.

## Настройка логирования

```python
# app/main.py
import logging
from fastapi import FastAPI, Request
from starlette.middleware.base import BaseHTTPMiddleware

# Настройка logging
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)-8s | %(name)s | %(message)s",
)

logger = logging.getLogger("app")

app = FastAPI()
```

## Middleware для логирования запросов

```python
import time
import logging
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware

logger = logging.getLogger("app.requests")

class RequestLoggingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        start = time.time()

        # Логирование запроса
        logger.info(
            f"{request.method} {request.url.path} - Start",
            extra={
                "method": request.method,
                "path": request.url.path,
                "query_params": str(request.query_params),
                "client_ip": request.client.host if request.client else None,
            }
        )

        response = await call_next(request)

        duration = time.time() - start

        # Логирование ответа
        logger.info(
            f"{request.method} {request.url.path} - {response.status_code} "
            f"({duration:.3f}s)",
            extra={
                "method": request.method,
                "path": request.url.path,
                "status_code": response.status_code,
                "duration": duration,
            }
        )

        # Добавление заголовка времени
        response.headers["X-Process-Time"] = f"{duration:.3f}s"
        return response

app.add_middleware(RequestLoggingMiddleware)
```

## JSON логирование (для продакшена)

```python
# pip install python-json-logger
import logging
from pythonjsonlogger import jsonlogger

def setup_json_logging():
    handler = logging.StreamHandler()
    formatter = jsonlogger.JsonFormatter(
        "%(asctime)s %(levelname)s %(name)s %(message)s",
        timestamp=True,
    )
    handler.setFormatter(formatter)

    logger = logging.getLogger("app")
    logger.addHandler(handler)
    logger.setLevel(logging.INFO)

# Структурированные логи
logger.info("Request completed", extra={
    "method": "GET",
    "path": "/users/1",
    "status_code": 200,
    "duration": 0.045,
    "user_id": 123,
})
# {"asctime": "2024-01-15T10:30:00", "levelname": "INFO", "method": "GET", ...}
```

## Prometheus мониторинг

```python
# pip install prometheus-client
from prometheus_client import Counter, Histogram, generate_latest, CONTENT_TYPE_LATEST
from fastapi import FastAPI, Request, Response
from starlette.middleware.base import BaseHTTPMiddleware
import time

# Метрики
REQUEST_COUNT = Counter(
    "http_requests_total",
    "Total HTTP requests",
    ["method", "endpoint", "status_code"],
)
REQUEST_DURATION = Histogram(
    "http_request_duration_seconds",
    "HTTP request duration in seconds",
    ["method", "endpoint"],
)

class PrometheusMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        start = time.time()

        response = await call_next(request)
        duration = time.time() - start

        # Метрики
        REQUEST_COUNT.labels(
            method=request.method,
            endpoint=request.url.path,
            status_code=response.status_code,
        ).inc()

        REQUEST_DURATION.labels(
            method=request.method,
            endpoint=request.url.path,
        ).observe(duration)

        return response

app.add_middleware(PrometheusMiddleware)

# Endpoint для скрейпинга
@app.get("/metrics")
async def metrics():
    return Response(generate_latest(), media_type=CONTENT_TYPE_LATEST)
```

### Prometheus конфигурация

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "fastapi"
    static_configs:
      - targets: ["localhost:8000"]
```

## Health Check и Readiness

```python
from fastapi import FastAPI
from sqlalchemy import text

app = FastAPI()

@app.get("/health")
async def health_check():
    """Liveness probe — приложение живо"""
    return {"status": "ok"}

@app.get("/health/ready")
async def readiness_check(db: AsyncSession = Depends(get_db)):
    """Readiness probe — приложение готово принимать запросы"""
    try:
        await db.execute(text("SELECT 1"))
        return {"status": "ready", "database": "connected"}
    except Exception as e:
        return {"status": "not ready", "database": str(e)}
```

## Отслеживание ошибок

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
import traceback
import logging

logger = logging.getLogger("app.errors")

@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    # Логирование ошибки
    logger.error(
        f"Unhandled exception: {exc}",
        extra={
            "method": request.method,
            "path": request.url.path,
            "traceback": traceback.format_exc(),
        }
    )

    return JSONResponse(
        status_code=500,
        content={"detail": "Internal server error"},
    )

# Специфичные обработчики
@app.exception_handler(ValidationError)
async def validation_error_handler(request: Request, exc: ValidationError):
    logger.warning(f"Validation error: {exc}")
    return JSONResponse(status_code=400, content={"detail": str(exc)})
```

## Sentry интеграция

```python
# pip install sentry-sdk
import sentry_sdk
from sentry_sdk.integrations.fastapi import FastApiIntegration

sentry_sdk.init(
    dsn="https://your-dsn@sentry.io/123",
    integrations=[FastApiIntegration()],
    traces_sample_rate=1.0,
)
```

## Best Practices

✅ **Используйте** структурированные логи (JSON) для продакшена
✅ **Добавляйте** `/metrics` endpoint для Prometheus
✅ **Добавляйте** `/health` и `/health/ready` для оркестраторов
✅ **Логируйте** метод, путь, статус код, время выполнения
✅ **Используйте** Sentry для отслеживания ошибок

❌ **Не логируйте** чувствительные данные (токены, пароли)
❌ **Не логируйте** тело запроса для больших payload
❌ **Не игнорируйте** ошибки — логируйте и обрабатывайте

## Ссылки

- [FastAPI Middleware](https://fastapi.tiangolo.com/tutorial/middleware/)
- [Prometheus клиент](https://github.com/prometheus/client_python)
- [Sentry Python SDK](https://docs.sentry.io/platforms/python/)

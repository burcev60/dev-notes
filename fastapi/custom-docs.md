# FastAPI — Кастомизация документации

## Описание

Кастомизация Swagger UI, ReDoc, описание схем ошибок и метаданных API.

## Настройка FastAPI

```python
from fastapi import FastAPI

app = FastAPI(
    title="My API",
    summary="API for managing users and posts",
    description="""
## Описание

Это REST API для управления пользователями и постами.

### Возможности:
- **CRUD** пользователей
- **CRUD** постов
- **Аутентификация** через JWT

### Версия: 1.0.0
""",
    version="1.0.0",
    docs_url="/docs",        # Swagger UI
    redoc_url="/redoc",      # ReDoc
    openapi_url="/openapi.json",
    openapi_tags=[
        {
            "name": "users",
            "description": "Операции с пользователями",
        },
        {
            "name": "posts",
            "description": "Операции с постами",
        },
    ],
)
```

## Кастомизация Swagger UI

```python
app = FastAPI(
    docs_url="/docs",
    swagger_ui_parameters={
        "defaultModelsExpandDepth": -1,  # Скрыть модели
        "displayRequestDuration": True,   # Показывать время запроса
        "filter": True,                   # Поиск
        "syntaxHighlight.theme": "monokai",
    },
)

# Отключение документации
app = FastAPI(docs_url=None, redoc_url=None)

# Кастомные URL
app = FastAPI(
    docs_url="/swagger",
    redoc_url="/api-docs",
    openapi_url="/api/openapi.json",
)
```

## Описание ответов (responses)

```python
from fastapi import status
from pydantic import BaseModel

class ErrorResponse(BaseModel):
    detail: str
    error_code: str | None = None

class ValidationErrorDetail(BaseModel):
    field: str
    message: str

class ValidationErrorResponse(BaseModel):
    detail: str
    errors: list[ValidationErrorDetail]

@app.post(
    "/users/",
    response_model=UserOut,
    status_code=status.HTTP_201_CREATED,
    responses={
        201: {
            "description": "User created successfully",
            "content": {
                "application/json": {
                    "example": {
                        "id": 1,
                        "username": "alice",
                        "email": "alice@example.com",
                        "is_active": True,
                    }
                }
            },
        },
        400: {
            "model": ErrorResponse,
            "description": "Bad request",
            "content": {
                "application/json": {
                    "examples": {
                        "email_exists": {
                            "summary": "Email already registered",
                            "value": {
                                "detail": "Email already registered",
                                "error_code": "EMAIL_EXISTS",
                            },
                        },
                        "username_taken": {
                            "summary": "Username already taken",
                            "value": {
                                "detail": "Username already taken",
                                "error_code": "USERNAME_TAKEN",
                            },
                        },
                    }
                }
            },
        },
        422: {
            "model": ValidationErrorResponse,
            "description": "Validation error",
        },
    },
)
async def create_user(data: UserCreate):
    ...
```

## OpenAPI схема

```python
# Кастомизация OpenAPI
from fastapi.openapi.utils import get_openapi

def custom_openapi():
    if app.openapi_schema:
        return app.openapi_schema

    openapi_schema = get_openapi(
        title="My API",
        version="1.0.0",
        routes=app.routes,
    )

    # Добавление security схем
    openapi_schema["components"]["securitySchemes"] = {
        "bearerAuth": {
            "type": "http",
            "scheme": "bearer",
            "bearerFormat": "JWT",
        }
    }

    # Глобальная security
    # openapi_schema["security"] = [{"bearerAuth": []}]

    app.openapi_schema = openapi_schema
    return app.openapi_schema

app.openapi = custom_openapi
```

## Теги для роутеров

```python
# Группировка эндпоинтов
users_router = APIRouter(
    prefix="/users",
    tags=["users"],  # Тег в Swagger
)

posts_router = APIRouter(
    prefix="/posts",
    tags=["posts"],
)

# Тег с описанием
app.openapi_tags = [
    {"name": "users", "description": "User management"},
    {"name": "posts", "description": "Post management"},
    {"name": "auth", "description": "Authentication endpoints"},
]
```

## Примеры в схемах

```python
from pydantic import BaseModel, Field

class UserCreate(BaseModel):
    username: str = Field(
        ...,
        min_length=3,
        max_length=50,
        examples=["alice", "bob123"],
    )
    email: str = Field(
        ...,
        examples=["alice@example.com"],
    )
    password: str = Field(
        ...,
        min_length=8,
        examples=["MySecureP@ss123"],
        format="password",
        json_schema_extra={"writeOnly": True},
    )

    model_config = {
        "json_schema_extra": {
            "examples": [
                {
                    "username": "alice",
                    "email": "alice@example.com",
                    "password": "MySecureP@ss123",
                }
            ]
        }
    }
```

## Заголовки в ответах

```python
from fastapi import Response

@app.get("/items/{item_id}")
async def get_item(item_id: int, response: Response):
    response.headers["X-RateLimit-Limit"] = "100"
    response.headers["X-RateLimit-Remaining"] = "99"

    return {"id": item_id, "name": "Item"}
```

## Best Practices

✅ **Описывайте** все возможные ответы (200, 400, 404, 422, 500)
✅ **Используйте** `examples` для наглядности
✅ **Добавляйте** теги для группировки
✅ **Добавляйте** описания к полям и эндпоинтам
✅ **Используйте** `status_code` для корректного ответа

❌ **Не отключайте** документацию в dev/staging
❌ **Не забывайте** про описание ошибок
❌ **Не используйте** дефолтные описания — пишите осмысленные

## Ссылки

- [FastAPI OpenAPI](https://fastapi.tiangolo.com/advanced/extending-openapi/)
- [Swagger UI параметры](https://swagger.io/docs/open-source-tools/swagger-ui/usage/configuration/)
- [ReDoc](https://github.com/Redocly/redoc)

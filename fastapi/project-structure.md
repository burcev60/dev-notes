# FastAPI — Структура проекта

## Описание

Рекомендуемая структура FastAPI проекта с разделением на модули: routers, schemas, services, db.

## Структура проекта

```
my-fastapi-app/
├── pyproject.toml
├── uv.lock
├── .env
├── .env.example
├── alembic.ini
├── alembic/
│   ├── env.py
│   ├── script.py.mako
│   └── versions/
│       └── 001_create_users.py
├── app/
│   ├── __init__.py
│   ├── main.py                 # Точка входа, создание app
│   ├── config.py               # Settings (pydantic-settings)
│   ├── dependencies.py         # Общие зависимости (get_db, get_current_user)
│   ├── models/                 # SQLAlchemy модели
│   │   ├── __init__.py
│   │   ├── base.py             # DeclarativeBase
│   │   ├── user.py
│   │   └── post.py
│   ├── schemas/                # Pydantic схемы (request/response)
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── post.py
│   ├── services/               # Бизнес-логика
│   │   ├── __init__.py
│   │   ├── user_service.py
│   │   └── post_service.py
│   ├── api/                    # Роутеры
│   │   ├── __init__.py
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── router.py       # Объединение v1 роутеров
│   │   │   ├── users.py
│   │   │   └── posts.py
│   │   └── dependencies.py     # API-specific зависимости
│   └── core/                   # Утилиты, security
│       ├── __init__.py
│       ├── security.py         # JWT, password hashing
│       └── exceptions.py       # Кастомные исключения
└── tests/
    ├── __init__.py
    ├── conftest.py             # Fixtures
    ├── test_users.py
    └── test_posts.py
```

## main.py — Точка входа

```python
# app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.config import settings
from app.api.v1.router import router as v1_router

def create_app() -> FastAPI:
    app = FastAPI(
        title=settings.app_name,
        version=settings.app_version,
        docs_url="/docs",
        redoc_url="/redoc",
    )

    # Middleware
    app.add_middleware(
        CORSMiddleware,
        allow_origins=settings.cors_origins,
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )

    # Роуты
    app.include_router(v1_router, prefix="/api/v1")

    # Health check
    @app.get("/health")
    async def health_check():
        return {"status": "ok"}

    return app

app = create_app()
```

## config.py — Настройки

```python
# app/config.py
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    app_name: str = "My API"
    app_version: str = "1.0.0"
    debug: bool = False

    # Database
    database_url: str = "postgresql+asyncpg://user:pass@localhost/db"
    db_pool_size: int = 10
    db_max_overflow: int = 20

    # Security
    secret_key: str = "changeme"
    access_token_expire_minutes: int = 30

    # CORS
    cors_origins: list[str] = ["http://localhost:3000"]

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        env_nested_delimiter="__",
    )

settings = Settings()
```

## models/base.py

```python
# app/models/base.py
from sqlalchemy.orm import DeclarativeBase

class Base(DeclarativeBase):
    pass
```

## models/user.py

```python
# app/models/user.py
from sqlalchemy import String, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column
from app.models.base import Base
from datetime import datetime

class User(Base):
    __tablename__ = "users"
    __table_args__ = {"extend_existing": True}

    id: Mapped[int] = mapped_column(primary_key=True)
    username: Mapped[str] = mapped_column(String(50), unique=True, index=True)
    email: Mapped[str] = mapped_column(String(100), unique=True)
    hashed_password: Mapped[str] = mapped_column(String(255))
    is_active: Mapped[bool] = mapped_column(default=True)
    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), server_default=func.now()
    )
```

## schemas/user.py

```python
# app/schemas/user.py
from pydantic import BaseModel, EmailStr, Field
from datetime import datetime

# Input — для создания
class UserCreate(BaseModel):
    username: str = Field(..., min_length=3, max_length=50)
    email: EmailStr
    password: str = Field(..., min_length=8)

# Input — для обновления (все поля опциональны)
class UserUpdate(BaseModel):
    username: str | None = None
    email: EmailStr | None = None
    password: str | None = None

# Output — для ответа (без пароля)
class UserOut(BaseModel):
    id: int
    username: str
    email: str
    is_active: bool
    created_at: datetime

    model_config = {"from_attributes": True}
```

## services/user_service.py

```python
# app/services/user_service.py
from sqlalchemy import select, func
from sqlalchemy.ext.asyncio import AsyncSession
from app.models.user import User
from app.core.security import get_password_hash

class UserService:
    def __init__(self, db: AsyncSession):
        self.db = db

    async def get_by_id(self, user_id: int) -> User | None:
        return await self.db.get(User, user_id)

    async def get_by_email(self, email: str) -> User | None:
        stmt = select(User).where(User.email == email)
        result = await self.db.execute(stmt)
        return result.scalar_one_or_none()

    async def create(self, data: dict) -> User:
        user = User(
            username=data["username"],
            email=data["email"],
            hashed_password=get_password_hash(data["password"]),
        )
        self.db.add(user)
        await self.db.flush()
        await self.db.refresh(user)
        return user
```

## api/v1/users.py

```python
# app/api/v1/users.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession
from app.db.session import get_db
from app.schemas.user import UserCreate, UserOut, UserUpdate
from app.services.user_service import UserService
from app.dependencies import get_user_service

router = APIRouter(prefix="/users", tags=["users"])

@router.post("/", response_model=UserOut, status_code=status.HTTP_201_CREATED)
async def create_user(
    data: UserCreate,
    service: UserService = Depends(get_user_service),
):
    existing = await service.get_by_email(data.email)
    if existing:
        raise HTTPException(status_code=400, detail="Email already registered")
    return await service.create(data.model_dump())

@router.get("/{user_id}", response_model=UserOut)
async def get_user(user_id: int, service: UserService = Depends(get_user_service)):
    user = await service.get_by_id(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

## api/v1/router.py

```python
# app/api/v1/router.py
from fastapi import APIRouter
from app.api.v1.users import router as users_router
from app.api.v1.posts import router as posts_router

router = APIRouter()

router.include_router(users_router)
router.include_router(posts_router)
```

## dependencies.py

```python
# app/dependencies.py
from collections.abc import AsyncGenerator
from sqlalchemy.ext.asyncio import AsyncSession
from app.db.session import AsyncSession as SessionFactory
from app.services.user_service import UserService

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with SessionFactory() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise

async def get_user_service(db: AsyncSession = Depends(get_db)):
    return UserService(db)
```

## Запуск проекта

```bash
# Development
uvicorn app.main:app --reload

# Production
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

## Best Practices

✅ **Разделяйте** models/schemas/services/router
✅ **Используйте** сервисный слой для бизнес-логики
✅ **Используйте** отдельные схемы для input/output
✅ **Используйте** `from_attributes=True` в output схемах
✅ **Используйте** `APIRouter` с тегами для организации
✅ **Используйте** `status.HTTP_*` константы

❌ **Не возвращайте** модели БД напрямую (используйте schemas)
❌ **Не смешивайте** бизнес-логику с роутерами
❌ **Не создавайте** все роуты в одном файле
❌ **Не забывайте** `__init__.py` в пакетах

## Ссылки

- [FastAPI Bigger Applications](https://fastapi.tiangolo.com/tutorial/bigger-applications/)
- [FastAPI Project Structure](https://fastapi.tiangolo.com/project-generation/)

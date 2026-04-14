# FastAPI + Pydantic модели

## Описание

Pydantic v2 — основа валидации данных в FastAPI. Request/response модели, автоматическая генерация JSON Schema.

## Request модели

```python
from pydantic import BaseModel, Field, EmailStr
from datetime import datetime

class UserCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=50)
    email: EmailStr
    age: int = Field(..., ge=0, le=150)
    tags: list[str] = []

@app.post("/users/")
async def create_user(user: UserCreate):
    return user
```

## Response модели

```python
class UserOut(BaseModel):
    id: int
    name: str
    email: str
    created_at: datetime

    model_config = ConfigDict(from_attributes=True)

@app.get("/users/{user_id}", response_model=UserOut)
async def get_user(user_id: int):
    user = db.get_user(user_id)
    return user  # Автоматическая сериализация
```

## Вложенные модели

```python
class Address(BaseModel):
    city: str
    street: str
    zip: str

class UserWithAddress(BaseModel):
    name: str
    email: str
    address: Address

@app.post("/users/", response_model=UserWithAddress)
async def create_user(user: UserWithAddress):
    return user
```

## Наследование моделей

```python
class UserBase(BaseModel):
    name: str
    email: EmailStr

class UserCreate(UserBase):
    password: str

class UserUpdate(UserBase):
    password: str | None = None

class UserOut(UserBase):
    id: int
    is_active: bool

    model_config = ConfigDict(from_attributes=True)
```

## Валидация

```python
from pydantic import field_validator, model_validator

class User(BaseModel):
    password: str
    confirm_password: str

    @field_validator('password')
    @classmethod
    def password_strength(cls, v: str) -> str:
        if len(v) < 8:
            raise ValueError('Password must be at least 8 characters')
        return v

    @model_validator(mode='before')
    @classmethod
    def passwords_match(cls, data: dict) -> dict:
        if data.get('password') != data.get('confirm_password'):
            raise ValueError('Passwords do not match')
        return data
```

## Best Practices

✅ **Разделяйте** input/output модели
✅ **Используйте** `from_attributes=True` для ORM
✅ **Используйте** `EmailStr`, `HttpUrl` и т.д.

❌ **Не используйте** одну модель для всего
❌ **Не возвращайте** password в response

## Ссылки

- [Pydantic документация](https://docs.pydantic.dev/)
- [FastAPI схемы](https://fastapi.tiangolo.com/tutorial/response-model/)

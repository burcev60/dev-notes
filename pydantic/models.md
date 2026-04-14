# Pydantic — BaseModel и валидация

## Описание

Pydantic v2 — библиотека для валидации данных и сериализации через модели.

## Базовая модель

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    name: str = Field(..., min_length=1, max_length=50)
    email: str
    age: int = Field(..., ge=0, le=150)
    is_active: bool = True
```

## Валидация

```python
from pydantic import field_validator, EmailStr

class User(BaseModel):
    email: EmailStr

    @field_validator('name')
    @classmethod
    def name_upper(cls, v: str) -> str:
        return v.upper()
```

## Best Practices

✅ **Используйте** `Field()` для ограничений
✅ **Используйте** валидаторы для сложной логики

## Ссылки

- [Pydantic документация](https://docs.pydantic.dev/)

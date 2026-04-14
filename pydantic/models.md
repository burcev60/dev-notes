# Pydantic — BaseModel и валидация

## Описание

Pydantic v2 — библиотека для валидации данных и сериализации через модели. Построена на `pydantic-core` (Rust), в 5-50 раз быстрее v1.

**Версия:** pydantic 2.13.0, pydantic-core 2.46.0

## Базовая модель

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    id: int
    name: str = Field(..., min_length=1, max_length=50)
    email: str
    age: int = Field(..., ge=0, le=150)
    is_active: bool = True
    tags: list[str] = []

# Создание
user = User(id=1, name="Alice", email="alice@example.com", age=30)
print(user)  # User(id=1, name='Alice', email='alice@example.com', age=30, is_active=True, tags=[])

# Валидация при создании
# User(id=1, name="", email="bad", age=-1)  # ValidationError!
```

## Field — ограничения и мета

```python
from pydantic import Field, AliasChoices

class Product(BaseModel):
    name: str = Field(
        ...,
        min_length=1,
        max_length=100,
        description="Product name",
        examples=["Laptop Pro 15"],
    )
    price: float = Field(..., gt=0, le=1_000_000)
    quantity: int = Field(default=0, ge=0)
    sku: str = Field(
        ...,
        pattern=r"^[A-Z]{2}-\d{4}$",  # regex валидация
        examples=["AB-1234"],
    )
    # Альтернативные имена поля (aliases)
    category: str = Field(
        ...,
        validation_alias=AliasChoices("category", "cat", "type"),
    )
```

## Сетевые типы

```python
from pydantic import (
    EmailStr, HttpUrl, AnyUrl, IPvAnyAddress,
    PostgresDsn, RedisDsn, MySQLDsn,
)

class Config(BaseModel):
    admin_email: EmailStr
    api_url: HttpUrl  # Валидирует URL
    database: PostgresDsn
    cache: RedisDsn
    server_ip: IPvAnyAddress

config = Config(
    admin_email="admin@example.com",
    api_url="https://api.example.com/v1",
    database="postgresql://user:pass@localhost/db",
    cache="redis://localhost:6379/0",
    server_ip="192.168.1.1",
)
```

## Валидаторы полей

```python
from pydantic import field_validator, field_serializer
from pydantic import BeforeValidator, AfterValidator, PlainValidator
from pydantic import ValidationInfo
from datetime import datetime

class Order(BaseModel):
    email: str
    password: str
    created_at: datetime
    items: list[str]

    # field_validator — валидация одного поля
    @field_validator('email')
    @classmethod
    def validate_email(cls, v: str) -> str:
        if '@' not in v:
            raise ValueError('Invalid email')
        return v.lower()

    # Валидация с контекстом
    @field_validator('password')
    @classmethod
    def password_strength(cls, v: str, info: ValidationInfo) -> str:
        if len(v) < 8:
            raise ValueError('Password must be at least 8 characters')
        if not any(c.isupper() for c in v):
            raise ValueError('Password must contain uppercase letter')
        return v

    # Несколько полей
    @field_validator('items')
    @classmethod
    def validate_items(cls, v: list[str]) -> list[str]:
        if not v:
            raise ValueError('At least one item required')
        return v
```

## Model валидаторы

```python
from pydantic import model_validator, ValidationInfo

class User(BaseModel):
    password: str
    confirm_password: str
    birth_date: datetime
    age: int

    # before — до валидации полей
    @model_validator(mode='before')
    @classmethod
    def preprocess(cls, data: dict) -> dict:
        if isinstance(data, dict):
            data['email'] = data.get('email', '').lower().strip()
        return data

    # after — после валидации всех полей
    @model_validator(mode='after')
    def passwords_match(self) -> 'User':
        if self.password != self.confirm_password:
            raise ValueError('Passwords do not match')
        return self

    # with_info — доступ к контексту
    @model_validator(mode='after')
    def age_matches_birthdate(self, info: ValidationInfo) -> 'User':
        # info.context — передаётся при валидации
        if self.birth_date and self.age:
            expected_age = datetime.now().year - self.birth_date.year
            if abs(expected_age - self.age) > 1:
                raise ValueError('Age does not match birth date')
        return self
```

## Функциональные валидаторы

```python
from pydantic import BeforeValidator, AfterValidator, PlainValidator, WrapValidator
from typing import Annotated

# BeforeValidator — преобразование до валидации
def strip_whitespace(v: str) -> str:
    return v.strip() if isinstance(v, str) else v

Name = Annotated[str, BeforeValidator(strip_whitespace)]

# AfterValidator — проверка после валидации
def positive(v: int) -> int:
    if v <= 0:
        raise ValueError('Must be positive')
    return v

PositiveInt = Annotated[int, AfterValidator(positive)]

# PlainValidator — полная замена валидации
def parse_int(v) -> int:
    try:
        return int(v)
    except (ValueError, TypeError):
        raise ValueError('Must be convertible to int')

IntFromString = Annotated[int, PlainValidator(parse_int)]

# WrapValidator — обёртка вокруг стандартной валидации
from pydantic import ValidatorFunctionWrapHandler

def wrap_int(v: str, handler: ValidatorFunctionWrapHandler) -> int:
    result = handler(v)  # Стандартная валидация
    return result * 2  # Модификация

IntDoubled = Annotated[int, WrapValidator(wrap_int)]

# Использование
class Product(BaseModel):
    name: Name
    price: PositiveInt
```

## computed_field

```python
from pydantic import computed_field

class Rectangle(BaseModel):
    width: float
    height: float

    @computed_field
    @property
    def area(self) -> float:
        return self.width * self.height

    @computed_field(alias="perimeter_value")
    @property
    def perimeter(self) -> float:
        return 2 * (self.width + self.height)

rect = Rectangle(width=5, height=10)
print(rect.area)       # 50.0
print(rect.model_dump())
# {'width': 5.0, 'height': 10.0, 'area': 50.0, 'perimeter_value': 30.0}
```

## ConfigDict

```python
from pydantic import BaseModel, ConfigDict

class User(BaseModel):
    model_config = ConfigDict(
        from_attributes=True,      # Конвертация из ORM
        str_strip_whitespace=True, # Автоматический strip строк
        str_min_length=1,          # Мин. длина строки
        validate_default=True,     # Валидация default значений
        extra='forbid',            # 'allow' | 'ignore' | 'forbid'
        frozen=True,               # Immutable модель
        json_schema_extra={        # Доп. мета для JSON Schema
            "examples": [{"name": "Alice"}]
        },
    )
    name: str
    email: str
```

## TypeAdapter — валидация без модели

```python
from pydantic import TypeAdapter
from typing import list

# Валидация произвольного типа
IntAdapter = TypeAdapter(int)
IntAdapter.validate_python("42")  # 42

ListStr = TypeAdapter(list[str])
ListStr.validate_python(["a", "b", "c"])

# JSON валидация
JsonAdapter = TypeAdapter(dict)
result = JsonAdapter.validate_json('{"key": "value"}')

# JSON Schema
schema = ListStr.json_schema()
```

## RootModel

```python
from pydantic import RootModel

# Модель с одним корневым полем
class UserList(RootModel[list[User]]):
    pass

users = UserList.model_validate([
    {"id": 1, "name": "Alice", "email": "a@x.com", "age": 30},
    {"id": 2, "name": "Bob", "email": "b@x.com", "age": 25},
])

for user in users.root:
    print(user.name)
```

## validate_call — валидация функций

```python
from pydantic import validate_call

@validate_call
def create_user(
    name: str,
    email: str,
    age: int = Field(ge=0, le=150),
) -> dict:
    return {"name": name, "email": email, "age": age}

create_user("Alice", "alice@example.com", 30)
# create_user("Bob", "not-an-email", 30)  # ValidationError!
```

## Best Practices

✅ **Используйте** `Field()` для ограничений и meta
✅ **Используйте** `@field_validator` для сложной логики
✅ **Используйте** `model_validator(mode='after')` для кросс-полевых проверок
✅ **Используйте** `TypeAdapter` для валидации без модели
✅ **Используйте** `extra='forbid'` для строгих схем

❌ **Не используйте** `root_validator` (deprecated) — используйте `model_validator`
❌ **Не храните** бизнес-логику в моделях — только валидация
❌ **Не игнорируйте** ValidationError — логируйте детали

## Ссылки

- [Pydantic документация](https://docs.pydantic.dev/)
- [Pydantic Fields](https://docs.pydantic.dev/latest/concepts/fields/)
- [Pydantic Validators](https://docs.pydantic.dev/latest/concepts/validators/)

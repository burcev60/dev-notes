# Pydantic — Сериализация

## Описание

Методы для преобразования моделей в dict, JSON и обратно.

## model_dump — в dict

```python
from pydantic import BaseModel, computed_field, SecretStr

class User(BaseModel):
    id: int
    name: str
    email: str
    password: SecretStr
    age: int | None = None

    @computed_field
    @property
    def is_adult(self) -> bool:
        return self.age is not None and self.age >= 18

user = User(id=1, name="Alice", email="a@x.com", password="secret123", age=30)

# Базовый dump
data = user.model_dump()
# {'id': 1, 'name': 'Alice', 'email': 'a@x.com', 'password': SecretStr('**********'), 'age': 30, 'is_adult': True}

# Исключение полей
data = user.model_dump(exclude={'password'})
# {'id': 1, 'name': 'Alice', 'email': 'a@x.com', 'age': 30, 'is_adult': True}

# Включение только определённых полей
data = user.model_dump(include={'id', 'name'})
# {'id': 1, 'name': 'Alice'}

# Режим сериализации
data = user.model_dump(mode='python')  # Python объекты (default)
data = user.model_dump(mode='json')    # JSON-сериализуемые значения
# {'password': '**********', ...}
```

## model_dump_json — в JSON строку

```python
# JSON строка
json_str = user.model_dump_json()
# '{"id":1,"name":"Alice","email":"a@x.com","password":"**********","age":30,"is_adult":true}'

# С исключениями
json_str = user.model_dump_json(exclude={'password'})

# С indent (pretty print)
json_str = user.model_dump_json(indent=2)
```

## model_validate — из dict

```python
# Из dict
user = User.model_validate({
    "id": 1,
    "name": "Bob",
    "email": "b@x.com",
    "password": "pass456",
    "age": 25,
})

# Строгая валидация (strict=True — без конверсии типов)
# User.model_validate({"id": "1", ...}, strict=True)  # error: "1" не int
```

## model_validate_json — из JSON

```python
user = User.model_validate_json(
    '{"id": 1, "name": "Charlie", "email": "c@x.com", "password": "pass", "age": 40}'
)
```

## model_copy — копирование

```python
user2 = user.model_copy()
user2 = user.model_copy(update={"name": "Alice Updated"})
user2 = user.model_copy(deep=True)  # Глубокая копия
```

## Сериализаторы

```python
from pydantic import BaseModel, field_serializer, model_serializer
from datetime import datetime

class Article(BaseModel):
    id: int
    title: str
    created_at: datetime
    tags: list[str]

    # Полевой сериализатор
    @field_serializer('created_at')
    def serialize_dt(self, value: datetime) -> str:
        return value.strftime('%Y-%m-%d %H:%M')

    @field_serializer('tags')
    def serialize_tags(self, value: list[str]) -> str:
        return ", ".join(value)

    # Модельный сериализатор
    @model_serializer(mode='wrap')
    def serialize_model(self, handler, info) -> dict:
        result = handler(self)
        result['_type'] = 'article'
        return result

article = Article(id=1, title="Hello", created_at=datetime.now(), tags=["python", "pydantic"])
print(article.model_dump())
# {'id': 1, 'title': 'Hello', 'created_at': '2024-01-15 14:30', 'tags': 'python, pydantic', '_type': 'article'}
```

## JSON Schema

```python
# Схема всей модели
schema = User.model_json_schema()

# Схема с ref template
schema = User.model_json_schema(
    ref_template="#/components/schemas/{model}"
)

# Schema mode
schema = User.model_json_schema(mode='serialization')
schema = User.model_json_schema(mode='validation')

# Сериализация схемы
import json
print(json.dumps(schema, indent=2))
```

## TypeAdapter сериализация

```python
from pydantic import TypeAdapter
from datetime import datetime

DateAdapter = TypeAdapter(datetime)

# В JSON
json_str = DateAdapter.dump_json(datetime.now())

# Из JSON
dt = DateAdapter.validate_json('"2024-01-15T14:30:00"')
```

## from_attributes — конвертация из ORM

```python
from pydantic import BaseModel, ConfigDict

class UserOut(BaseModel):
    model_config = ConfigDict(from_attributes=True)

    id: int
    name: str
    email: str

# SQLAlchemy модель
# class User(Base):
#     id = mapped_column(Integer, primary_key=True)
#     name = mapped_column(String)
#     email = mapped_column(String)

# Конвертация
user_orm = session.get(User, 1)
user_out = UserOut.model_validate(user_orm)  # Работает!
```

## Best Practices

✅ **Используйте** `model_dump(exclude={...})` для фильтрации
✅ **Используйте** `from_attributes=True` для ORM → Pydantic
✅ **Используйте** `@field_serializer` для кастомного формата
✅ **Используйте** `SecretStr` для чувствительных данных

❌ **Не сериализуйте** пароли и токены в JSON
❌ **Не используйте** `dict(model)` — используйте `model.model_dump()`
❌ **Не забывайте** `mode='json'` при сериализации для API

## Ссылки

- [Pydantic Serialization](https://docs.pydantic.dev/latest/concepts/serialization/)
- [Pydantic JSON Schema](https://docs.pydantic.dev/latest/concepts/json_schema/)

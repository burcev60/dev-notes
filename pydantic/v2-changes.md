# Pydantic — Отличия v1 → v2

## Описание

Ключевые изменения в Pydantic v2. Переписан на Rust (pydantic-core), в 5-50 раз быстрее.

**Версия:** pydantic 2.13.0, pydantic-core 2.46.0

## Переименования методов

| Pydantic v1 | Pydantic v2 |
|-------------|-------------|
| `.dict()` | `.model_dump()` |
| `.json()` | `.model_dump_json()` |
| `.parse_obj(data)` | `.model_validate(data)` |
| `.parse_raw(json_str)` | `.model_validate_json(json_str)` |
| `.construct()` | `.model_construct()` |
| `.copy()` | `.model_copy()` |
| `.schema()` | `.model_json_schema()` |
| `MyModel.__fields__` | `MyModel.model_fields` |
| `MyModel.__config__` | `MyModel.model_config` |

## Конфигурация

```python
# v1
class Config:
    orm_mode = True
    allow_population_by_field_name = True

# v2
model_config = ConfigDict(
    from_attributes=True,           # вместо orm_mode
    populate_by_name=True,          # вместо allow_population_by_field_name
)
```

## Валидаторы

```python
# v1
from pydantic import validator, root_validator

@validator('field')
def validate_field(cls, v):
    return v

@root_validator
def validate_all(cls, values):
    return values

# v2
from pydantic import field_validator, model_validator

@field_validator('field')
@classmethod
def validate_field(cls, v):
    return v

@model_validator(mode='before')
@classmethod
def preprocess(cls, data):
    return data

@model_validator(mode='after')
def validate_all(self):
    return self
```

## Полевые валидаторы

```python
# v2 — функциональные валидаторы (новое!)
from pydantic import BeforeValidator, AfterValidator, PlainValidator, WrapValidator
from typing import Annotated

PositiveInt = Annotated[int, AfterValidator(lambda x: x if x > 0 else (_ for _ in ()).throw(ValueError()))]

class Model(BaseModel):
    value: PositiveInt
```

## Декораторы сериализации

```python
# v1
from pydantic import serializer

@serializer('field')
def serialize_field(cls, v):
    return v

# v2
from pydantic import field_serializer, model_serializer

@field_serializer('field')
def serialize_field(self, value):
    return value

@model_serializer(mode='wrap')
def serialize_model(self, handler, info):
    return handler(self)
```

## class_validators → model_validator

```python
# v1 root_validator(pre=True)
@root_validator(pre=True)
def preprocess(cls, values):
    return values

# v2
@model_validator(mode='before')
@classmethod
def preprocess(cls, data):
    return data

# v1 root_validator(skip_on_failure=True)
@root_validator(skip_on_failure=True)
def postprocess(cls, values):
    return values

# v2
@model_validator(mode='after')
def postprocess(self):
    return self
```

## TypeAdapter (новое в v2)

```python
# v2 — валидация без модели
from pydantic import TypeAdapter

IntList = TypeAdapter(list[int])
IntList.validate_python([1, 2, 3])
IntList.validate_json('[1, 2, 3]')
IntList.dump_python([1, 2, 3])
IntList.dump_json([1, 2, 3])
```

## RootModel (новое в v2)

```python
# v2
from pydantic import RootModel

class StringList(RootModel[list[str]]):
    pass

strings = StringList.model_validate(["a", "b", "c"])
for s in strings.root:
    print(s)
```

## validate_call (новое в v2)

```python
# v2 — валидация аргументов функции
from pydantic import validate_call

@validate_call
def create_user(name: str, age: int) -> dict:
    return {"name": name, "age": age}
```

## Изменения в типах

```python
# v1
from pydantic import PyObject

# v2
from pydantic import ImportString

class Config(BaseModel):
    module: ImportString  # вместо PyObject
```

## Производительность

| Операция | v1 | v2 | Ускорение |
|----------|----|----|-----------|
| Создание модели | 1x | 5-50x | Значительно |
| Валидация | 1x | 7-30x | Значительно |
| Сериализация | 1x | 3-10x | Значительно |

## Миграция

```bash
# Установка утилиты миграции
pip install pydantic-migrate

# Автоматическая миграция
pydantic-migrate migrate my_project/

# Проверка совместимости
pydantic-migrate check my_project/
```

## Best Practices

✅ **Используйте** `pydantic-migrate` для автоматической миграции
✅ **Проверяйте** все вызовы методов при обновлении
✅ **Тестируйте** валидацию после миграции
✅ **Используйте** TypeAdapter для простой валидации

❌ **Не смешивайте** v1 и v2 API в одном проекте
❌ **Не игнорируйте** deprecation warnings

## Ссылки

- [Pydantic v2 Migration Guide](https://docs.pydantic.dev/latest/migration/)
- [Pydantic v2 Blog Post](https://docs.pydantic.dev/latest/blog/pydantic-v2/)

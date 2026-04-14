# Pydantic — Отличия v1 → v2

## Описание

Ключевые изменения в Pydantic v2.

## Основные изменения

```python
# v1 → v2
# .dict() → .model_dump()
# .json() → .model_dump_json()
# .parse_obj() → .model_validate()
# .parse_raw() → .model_validate_json()
# class Config → model_config

# Валидаторы
# v1: @validator('field')
# v2: @field_validator('field')

# v1: root_validator
# v2: @model_validator(mode='before')
```

## Производительность

v2 переписан на Rust, в 5-50 раз быстрее.

## Best Practices

✅ **Обновите** вызовы методов при миграции
✅ **Используйте** `pydantic-migrate` для проверки

## Ссылки

- [Pydantic v2 Migration Guide](https://docs.pydantic.dev/latest/migration/)

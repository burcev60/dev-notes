# Pydantic — Сериализация

## Описание

Методы для преобразования моделей в dict, JSON и обратно.

## Сериализация

```python
user = User(name="Alice", email="a@x.com")

# В dict
user.model_dump()  # {'name': 'Alice', ...}

# В JSON
user.model_dump_json()  # '{"name": "Alice", ...}'

# С включением exclude
user.model_dump(exclude={'is_active'})
```

## Десериализация

```python
# Из dict
user = User.model_validate({"name": "Bob", "email": "b@x.com"})

# Из JSON
user = User.model_validate_json('{"name": "Charlie", "email": "c@x.com"}')
```

## Best Practices

✅ **Используйте** `model_validate()` для парсинга
✅ **Используйте** `exclude` для фильтрации

## Ссылки

- [Pydantic Serialization](https://docs.pydantic.dev/latest/concepts/serialization/)

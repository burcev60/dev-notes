# json — Сериализация JSON

## Описание

Модуль `json` предоставляет функции для кодирования и декодирования данных в формате JSON.

## Базовое использование

```python
import json

# Python dict → JSON строка (сериализация)
data = {'name': 'Alice', 'age': 30, 'active': True}
json_str = json.dumps(data)
print(json_str)  # {"name": "Alice", "age": 30, "active": true}

# JSON строка → Python dict (десериализация)
parsed = json.loads(json_str)
print(parsed['name'])  # 'Alice'

# Запись в файл
with open('data.json', 'w') as f:
    json.dump(data, f)

# Чтение из файла
with open('data.json', 'r') as f:
    data = json.load(f)
```

## Форматированный вывод

```python
import json

data = {'users': [{'name': 'Alice'}, {'name': 'Bob'}]}

# Красивый вывод с отступами
print(json.dumps(data, indent=2))
# {
#   "users": [
#     {"name": "Alice"},
#     {"name": "Bob"}
#   ]
# }

# Сортировка ключей
print(json.dumps(data, indent=2, sort_keys=True))

# Без ASCII эскейпинга (кириллица и др.)
data = {'message': 'Привет, мир!'}
print(json.dumps(data, ensure_ascii=False))
# {"message": "Привет, мир!"}
```

## Маппинг типов

| Python | JSON |
|--------|------|
| `dict` | `object` |
| `list`, `tuple` | `array` |
| `str` | `string` |
| `int`, `float` | `number` |
| `True` | `true` |
| `False` | `false` |
| `None` | `null` |

```python
import json

# Кортеж станет списком
print(json.dumps((1, 2, 3)))  # [1, 2, 3]

# None станет null
print(json.dumps(None))  # null
```

## Кастомная сериализация

```python
import json
from datetime import datetime

# Проблема: datetime не сериализуется по умолчанию
data = {'created': datetime.now()}
# json.dumps(data) → TypeError

# Решение 1: default функция
def custom_serializer(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    raise TypeError(f"Object of type {type(obj)} is not JSON serializable")

json_str = json.dumps(data, default=custom_serializer)

# Решение 2: cls параметр
class DateTimeEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        return super().default(obj)

json_str = DateTimeEncoder().encode(data)

# Решение 3: для Pydantic моделей
json_str = model.model_dump_json()
```

## Кастомная десериализация

```python
import json
from datetime import datetime

json_str = '{"created": "2024-01-15T14:30:00"}'

# object_pairs_hook — обработка при парсинге
def parse_dates(pairs):
    result = {}
    for key, value in pairs:
        if key.endswith('_at') or key == 'created':
            try:
                result[key] = datetime.fromisoformat(value)
            except (ValueError, TypeError):
                result[key] = value
        else:
            result[key] = value
    return result

data = json.loads(json_str, object_pairs_hook=parse_dates)
```

## Безопасность

```python
import json

# ⚠️ Никогда не используйте eval() для парсинга JSON!
# eval('{"name": "Alice"}')  # ОПАСНО!

# ✅ Используйте json.loads()
data = json.loads('{"name": "Alice"}')  # Безопасно

# Обработка ошибок
try:
    data = json.loads(invalid_json)
except json.JSONDecodeError as e:
    print(f"Invalid JSON: {e}")
```

## Best Practices

✅ **Используйте** `indent=2` для читаемости в dev/логах
✅ **Используйте** `ensure_ascii=False` для Unicode текста
✅ **Обрабатывайте** `json.JSONDecodeError` при парсинге
✅ **Используйте** Pydantic для сложной валидации

❌ **Не используйте** `eval()` для парсинга JSON
❌ **Не сериализуйте** чувствительные данные (пароли, токены)

## Ссылки

- [Официальная документация json](https://docs.python.org/3/library/json.html)
- [Pydantic — валидация и сериализация](https://docs.pydantic.dev/)

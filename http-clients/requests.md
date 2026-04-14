# requests — HTTP клиент

## Описание

`requests` — самая популярная HTTP библиотека для Python, простой синхронный HTTP клиент.

## Установка

```bash
pip install requests
```

## Базовое использование

```python
import requests

# GET запрос
response = requests.get("https://api.example.com/users")
print(response.status_code)  # 200
print(response.json())       # {'users': [...]}

# POST запрос
response = requests.post(
    "https://api.example.com/users",
    json={"name": "Alice", "email": "alice@example.com"}
)

# PUT, DELETE
response = requests.put("https://api.example.com/users/1", json={"name": "Updated"})
response = requests.delete("https://api.example.com/users/1")
```

## Параметры запроса

```python
# Query параметры
response = requests.get(
    "https://api.example.com/search",
    params={"q": "python", "page": 1, "limit": 10}
)
# URL: https://api.example.com/search?q=python&page=1&limit=10

# Заголовки
response = requests.get(
    "https://api.example.com/data",
    headers={"Authorization": "Bearer token123", "Accept": "application/json"}
)

# Timeout
response = requests.get("https://api.example.com", timeout=5)
```

## Обработка ответов

```python
response = requests.get("https://api.example.com")

# Status code
print(response.status_code)  # 200

# JSON
data = response.json()

# Текст
text = response.text

# Байты
content = response.content

# Проверка статуса
response.raise_for_status()  # Бросает HTTPError для 4xx/5xx
```

## Сессия

```python
# Session — переиспользование соединения
with requests.Session() as session:
    session.headers.update({"Authorization": "Bearer token"})
    session.get("https://api.example.com/users")
    session.get("https://api.example.com/posts")
    # Cookie сохраняются между запросами
```

## Best Practices

✅ **Используйте** `Session` для множества запросов
✅ **Всегда указывайте** `timeout`
✅ **Используйте** `raise_for_status()` для проверки

❌ **Не используйте** для async приложений
❌ **Не игнорируйте** ошибки HTTP

## Ссылки

- [Официальная документация requests](https://requests.readthedocs.io/)

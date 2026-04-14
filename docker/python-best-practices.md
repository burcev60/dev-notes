# Docker — Python Best Practices

## Описание

Рекомендации по контейнеризации Python приложений.

## Оптимизированный Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Кэш pip
ENV PIP_NO_CACHE_DIR=1

# Зависимости отдельно
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Код
COPY . .

# Непривилегированный пользователь
RUN useradd -m myuser && chown -R myuser:myuser /app
USER myuser

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Best Practices

✅ **Используйте** slim образы
✅ **Кэшируйте** слои (requirements до кода)
✅ **Запускайте** от непривилегированного пользователя
✅ **Используйте** `.dockerignore`

❌ **Не используйте** root в контейнере
❌ **Не включайте** dev зависимости в production

## Ссылки

- [Python Docker Best Practices](https://snyk.io/blog/best-practices-dockerfiles-python/)

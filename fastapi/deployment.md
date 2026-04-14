# FastAPI — Deployment

## Описание

Деплой FastAPI приложений с Uvicorn, Gunicorn для продакшена.

## Uvicorn

```bash
# Базовый запуск
uvicorn main:app --host 0.0.0.0 --port 8000

# Продакшен
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
```

## Gunicorn + Uvicorn

```bash
gunicorn main:app \
    -w 4 \
    -k uvicorn.workers.UvicornWorker \
    --bind 0.0.0.0:8000
```

## Docker

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Best Practices

✅ **Используйте** Gunicorn + Uvicorn для продакшена
✅ **Используйте** `--workers` для конкурентности

## Ссылки

- [FastAPI Deployment](https://fastapi.tiangolo.com/deployment/)

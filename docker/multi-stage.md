# Multi-stage builds

## Описание

Multi-stage builds позволяют уменьшить размер финального image.

## Пример

```dockerfile
# Stage 1: Build
FROM python:3.11 AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --prefix=/install -r requirements.txt

# Stage 2: Production
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /install /usr/local
COPY . .
CMD ["python", "main.py"]
```

## Best Practices

✅ **Используйте** для минимизации размера
✅ **Разделяйте** build и runtime зависимости

## Ссылки

- [Multi-stage builds](https://docs.docker.com/build/building/multi-stage/)

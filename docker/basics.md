# Docker — Основы

## Описание

Docker — платформа для контейнеризации приложений.

## Основные понятия

- **Image** — шаблон для создания контейнеров
- **Container** — запущенный экземпляр image
- **Dockerfile** — инструкция сборки

## Dockerfile

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "main.py"]
```

## Команды

```bash
# Сборка
docker build -t myapp:latest .

# Запуск
docker run -p 8000:8000 myapp

# Список контейнеров
docker ps

# Остановка
docker stop <container_id>
```

## Best Practices

✅ **Используйте** slim/alpine образы
✅ **Копируйте** requirements.txt отдельно от кода
✅ **Используйте** `.dockerignore`

## Ссылки

- [Docker документация](https://docs.docker.com/)

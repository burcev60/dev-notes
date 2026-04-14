# Docker Compose

## Описание

Docker Compose — инструмент для запуска multi-container приложений.

## docker-compose.yml

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## Команды

```bash
# Запуск
docker compose up -d

# Остановка
docker compose down

# Логи
docker compose logs -f app

# Пересборка
docker compose up -d --build
```

## Best Practices

✅ **Используйте** volumes для persistent данных
✅ **Используйте** `.env` для секретов

## Ссылки

- [Docker Compose документация](https://docs.docker.com/compose/)

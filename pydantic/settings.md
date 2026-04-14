# Pydantic — Settings

## Описание

BaseSettings для загрузки конфигурации из env и .env файлов.

## Использование

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    secret_key: str = "changeme"
    debug: bool = False

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8"
    )

settings = Settings()
```

## Best Practices

✅ **Используйте** `.env` для локальной разработки
✅ **Не храните** секреты в репозитории

## Ссылки

- [Pydantic Settings](https://docs.pydantic.dev/latest/concepts/pydantic_settings/)

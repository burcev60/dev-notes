# Pydantic — Settings

## Описание

BaseSettings для загрузки конфигурации из переменных окружения и `.env` файлов.

**Версия:** pydantic-settings 2.13.1

## Установка

```bash
pip install pydantic-settings
```

## Базовое использование

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    # Обычные поля с дефолтами
    app_name: str = "MyApp"
    debug: bool = False
    version: str = "1.0.0"

    # Секреты из окружения
    database_url: str
    secret_key: str
    api_token: str = ""

    # Конфигурация
    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        env_prefix="",           # Префикс переменных: APP_NAME
        env_nested_delimiter="__",  # Вложенность: DB__HOST
        extra="ignore",          # Игнорировать неизвестимые env
    )

settings = Settings()
```

## .env файл

```env
# .env
APP_NAME="My Production App"
DEBUG=false
DATABASE_URL="postgresql://user:pass@localhost/mydb"
SECRET_KEY="super-secret-key"
API_TOKEN="token123"
```

## Вложенные настройки

```python
class DatabaseSettings(BaseSettings):
    host: str = "localhost"
    port: int = 5432
    name: str = "mydb"
    user: str = ""
    password: str = ""

class Settings(BaseSettings):
    db: DatabaseSettings
    app_name: str = "MyApp"

    model_config = SettingsConfigDict(
        env_file=".env",
        env_nested_delimiter="__",
    )

# Переменные окружения:
# DB__HOST=db.example.com
# DB__PORT=5433
# DB__NAME=production
```

## Alias и префиксы

```python
from pydantic import Field, AliasChoices

class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_prefix="MY_APP_",  # MY_APP_DATABASE_URL
    )

    database_url: str = Field(
        ...,
        validation_alias=AliasChoices(
            "DATABASE_URL",       # Без префикса
            "MY_APP_DATABASE_URL", # С префиксом
            "DB_URL",             # Альтернатива
        ),
    )
```

## Источники значений (приоритет)

```python
# 1. Аргументы конструктора (высший приоритет)
# 2. Переменные окружения
# 3. .env файл
# 4. Default значения (низший приоритет)

settings = Settings(database_url="sqlite:///test.db")  # Переопределение
```

## Кастомные sources

```python
from pydantic_settings import (
    BaseSettings,
    SettingsConfigDict,
    PydanticBaseSettingsSource,
    SettingsSourceCallable,
)
import json

class JsonConfigSettingsSource(PydanticBaseSettingsSource):
    def get_field_value(self, field, field_name):
        pass  # Реализация чтения из JSON

    def __call__(self) -> dict:
        with open("config.json") as f:
            return json.load(f)

class Settings(BaseSettings):
    @classmethod
    def settings_customise_sources(
        cls,
        settings_cls: type[BaseSettings],
        init_settings: PydanticBaseSettingsSource,
        env_settings: PydanticBaseSettingsSource,
        dotenv_settings: PydanticBaseSettingsSource,
        file_secret_settings: PydanticBaseSettingsSource,
    ) -> tuple[SettingsSourceCallable, ...]:
        return (
            init_settings,
            JsonConfigSettingsSource(settings_cls),
            env_settings,
            dotenv_settings,
        )
```

## Best Practices

✅ **Используйте** `.env` для локальной разработки
✅ **Используйте** env variables для продакшена (Docker, CI/CD)
✅ **Используйте** `env_nested_delimiter` для группировки
✅ **Не храните** `.env` в репозитории

❌ **Не логируйте** secret_key, passwords
❌ **Не используйте** дефолтные секреты в продакшене

## Ссылки

- [Pydantic Settings](https://docs.pydantic.dev/latest/concepts/pydantic_settings/)
- [pydantic-settings GitHub](https://github.com/pydantic/pydantic-settings)

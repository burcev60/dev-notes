# Dependency Injection паттерны

## Описание

Dependency Injection (DI) — паттерн передачи зависимостей извне.

## Constructor Injection

```python
class UserService:
    def __init__(self, repository: UserRepository, email_service: EmailService):
        self.repository = repository
        self.email_service = email_service
```

## DI Container

```python
from dependency_injector import containers, providers

class Container(containers.DeclarativeContainer):
    config = providers.Configuration()

    db = providers.Singleton(Database, url=config.database_url)
    repository = providers.Singleton(UserRepository, db=db)
    email = providers.Singleton(EmailService)
    service = providers.Singleton(UserService, repository=repository, email_service=email)
```

## FastAPI DI

```python
def get_db() -> Generator[Session, None, None]:
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/users/")
def get_users(db: Session = Depends(get_db)):
    return db.query(User).all()
```

## Best Practices

✅ **Внедряйте** зависимости через конструктор
✅ **Используйте** DI контейнер для сложных графов
✅ **Используйте** FastAPI `Depends` для web

## Ссылки

- [Dependency Injector](https://python-dependency-injector.earnes.io/)

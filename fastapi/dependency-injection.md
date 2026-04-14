# FastAPI — Dependency Injection

## Описание

Система зависимостей (Dependencies) в FastAPI — мощный инструмент для переиспользуемой логики: аутентификация, пагинация, подключение к БД.

## Базовое использование

```python
from fastapi import Depends, FastAPI

app = FastAPI()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/users/")
def get_users(db = Depends(get_db)):
    return db.query(User).all()
```

## Зависимости с вложенностью

```python
def get_current_user(
    token: str = Depends(oauth2_scheme),
    db = Depends(get_db)
) -> User:
    user = db.query(User).filter(User.id == token.sub).first()
    if not user:
        raise HTTPException(status_code=401)
    return user

def get_active_user(
    current_user: User = Depends(get_current_user)
) -> User:
    if not current_user.is_active:
        raise HTTPException(status_code=400)
    return current_user

@app.get("/me/")
def read_me(user: User = Depends(get_active_user)):
    return user
```

## Переопределение зависимостей

```python
# Тестирование
from fastapi.testclient import TestClient

def override_get_db():
    try:
        db = TestingSessionLocal()
        yield db
    finally:
        db.close()

app.dependency_overrides[get_db] = override_get_db

client = TestClient(app)
response = client.get("/users/")
```

## yield зависимости

```python
async def get_current_user(token: str = Depends(oauth2_scheme)) -> User:
    # Pre-yield: получение
    credentials = verify_token(token)
    user = get_user_from_db(credentials.sub)

    yield user

    # Post-yield: cleanup (если нужен)
    log_access(user.id)
```

## Best Practices

✅ **Используйте** `Depends` для переиспользуемой логики
✅ **Используйте** yield для cleanup
✅ **Используйте** `dependency_overrides` для тестов

## Ссылки

- [FastAPI Dependencies](https://fastapi.tiangolo.com/tutorial/dependencies/)

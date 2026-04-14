# FastAPI — Аутентификация: OAuth2, JWT

## Описание

FastAPI предоставляет встроенные инструменты для OAuth2 и JWT аутентификации через `fastapi.security`.

## OAuth2 Password Bearer

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pydantic import BaseModel
from datetime import datetime, timedelta
import jwt  # PyJWT

app = FastAPI()

# Схема токена
SECRET_KEY = "your-secret-key"  # В продакшене — из Settings
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# OAuth2 схема — указывает endpoint для получения токена
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="auth/token")

# Модель пользователя
class User(BaseModel):
    username: str
    email: str
    is_active: bool = True

class Token(BaseModel):
    access_token: str
    token_type: str
```

## Создание JWT токена

```python
from datetime import datetime, timedelta
import jwt

def create_access_token(data: dict, expires_delta: timedelta | None = None):
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=30))
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

# Использование
token = create_access_token(
    data={"sub": "alice", "role": "admin"},
    expires_delta=timedelta(hours=1),
)
```

## Логин — получение токена

```python
from fastapi.security import OAuth2PasswordRequestForm

# Fake user database
fake_users = {
    "alice": {"username": "alice", "email": "alice@example.com", "hashed_password": "$2b$12$..."},
}

def verify_password(plain: str, hashed: str) -> bool:
    from passlib.context import CryptContext
    return CryptContext(schemes=["bcrypt"]).verify(plain, hashed)

@app.post("/auth/token", response_model=Token)
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = fake_users.get(form_data.username)
    if not user or not verify_password(form_data.password, user["hashed_password"]):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )

    token = create_access_token(
        data={"sub": user["username"], "role": "user"},
        expires_delta=timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES),
    )
    return {"access_token": token, "token_type": "bearer"}
```

## Защита эндпоинтов

```python
async def get_current_user(token: str = Depends(oauth2_scheme)) -> User:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
    except jwt.PyJWTError:
        raise credentials_exception

    user = fake_users.get(username)
    if user is None:
        raise credentials_exception
    return User(**user)

@app.get("/users/me")
async def read_users_me(current_user: User = Depends(get_current_user)):
    return current_user
```

## Роли и разрешения

```python
from functools import wraps

def require_role(role: str):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, current_user: User = Depends(get_current_user), **kwargs):
            payload = jwt.decode(
                Depends(oauth2_scheme), SECRET_KEY, algorithms=[ALGORITHM]
            )
            if payload.get("role") != role:
                raise HTTPException(status_code=403, detail="Forbidden")
            return await func(*args, current_user=current_user, **kwargs)
        return wrapper
    return decorator

# Использование
@app.get("/admin/panel")
@require_role("admin")
async def admin_panel(current_user: User):
    return {"message": f"Welcome, admin {current_user.username}"}
```

## OAuth2 Authorization Code Flow

```python
from fastapi.security import OAuth2AuthorizationCodeBearer

oauth2_code = OAuth2AuthorizationCodeBearer(
    authorizationUrl="https://auth.provider.com/authorize",
    tokenUrl="https://auth.provider.com/token",
)

# FastAPI автоматически добавляет кнопку "Authorize" в Swagger UI
@app.get("/protected")
async def protected(token: str = Depends(oauth2_code)):
    return {"token": token}
```

## HTTP Basic Auth

```python
from fastapi.security import HTTPBasic, HTTPBasicCredentials
import secrets

security = HTTPBasic()

@app.get("/basic-auth")
async def basic_auth(credentials: HTTPBasicCredentials = Depends(security)):
    correct_username = secrets.compare_digest(credentials.username, "admin")
    correct_password = secrets.compare_digest(credentials.password, "secret")
    if not (correct_username and correct_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect credentials",
            headers={"WWW-Authenticate": "Basic"},
        )
    return {"username": credentials.username}
```

## API Key

```python
from fastapi.security import APIKeyHeader, APIKeyQuery, APIKeyCookie
from fastapi import Security

api_key_header = APIKeyHeader(name="X-API-Key")
api_key_query = APIKeyQuery(name="api_key")
api_key_cookie = APIKeyCookie(name="api_key")

VALID_API_KEYS = {"key-123", "key-456"}

async def verify_api_key(
    header_key: str = Security(api_key_header),
    query_key: str = Security(api_key_query),
    cookie_key: str = Security(api_key_cookie),
):
    key = header_key or query_key or cookie_key
    if key not in VALID_API_KEYS:
        raise HTTPException(status_code=403, detail="Invalid API key")
    return key

@app.get("/api-key-protected")
async def api_key_protected(api_key: str = Depends(verify_api_key)):
    return {"api_key": api_key}
```

## Refresh Token

```python
class TokenPair(BaseModel):
    access_token: str
    refresh_token: str
    token_type: str

def create_refresh_token(data: dict) -> str:
    expire = datetime.utcnow() + timedelta(days=7)
    data.update({"exp": expire, "type": "refresh"})
    return jwt.encode(data, SECRET_KEY, algorithm=ALGORITHM)

@app.post("/auth/token", response_model=TokenPair)
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    # ... verification ...
    access = create_access_token({"sub": username})
    refresh = create_refresh_token({"sub": username})
    return {
        "access_token": access,
        "refresh_token": refresh,
        "token_type": "bearer",
    }

@app.post("/auth/refresh", response_model=Token)
async def refresh_token(refresh_token: str):
    try:
        payload = jwt.decode(refresh_token, SECRET_KEY, algorithms=[ALGORITHM])
        if payload.get("type") != "refresh":
            raise HTTPException(status_code=401, detail="Invalid token type")
    except jwt.PyJWTError:
        raise HTTPException(status_code=401, detail="Invalid refresh token")

    new_access = create_access_token({"sub": payload["sub"]})
    return {"access_token": new_access, "token_type": "bearer"}
```

## Best Practices

✅ **Храните** `SECRET_KEY` в环境变量/Settings
✅ **Используйте** `passlib` для хэширования паролей (bcrypt)
✅ **Устанавливайте** короткий TTL для access токенов (15-30 мин)
✅ **Используйте** refresh токены для долгосрочных сессий
✅ **Валидируйте** `sub` claim — это идентификатор пользователя
✅ **Используйте** `secrets.compare_digest()` для сравнения секретов

❌ **Не храните** секреты в коде — используйте `.env`
❌ **Не используйте** слабые секретные ключи
❌ **Не логируйте** токены
❌ **Не игнорируйте** `WWW-Authenticate` заголовок

## Ссылки

- [FastAPI Security](https://fastapi.tiangolo.com/tutorial/security/)
- [OAuth2 Password Bearer](https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/)
- [PyJWT документация](https://pyjwt.readthedocs.io/)
- [passlib](https://passlib.readthedocs.io/)

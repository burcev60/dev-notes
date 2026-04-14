# FastAPI — Основы

## Описание

FastAPI — современный асинхронный веб-фреймворк для Python, основанный на Pydantic и Starlette. Автоматическая генерация OpenAPI и Swagger документации.

## Установка

```bash
pip install fastapi uvicorn[standard]
```

## Базовое приложение

```python
from fastapi import FastAPI

app = FastAPI(
    title="My API",
    description="Example API",
    version="1.0.0"
)

@app.get("/")
async def root():
    return {"message": "Hello World"}

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    return {"user_id": user_id, "name": f"User {user_id}"}
```

## Запуск

```bash
uvicorn main:app --reload
# --reload — автоперезагрузка при изменении кода
```

## HTTP методы

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float
    is_offer: bool | None = None

@app.get("/items/")
async def read_items():
    return []

@app.post("/items/")
async def create_item(item: Item):
    return {"item": item.model_dump()}

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    return {"item_id": item_id, "item": item.model_dump()}

@app.delete("/items/{item_id}")
async def delete_item(item_id: int):
    return {"item_id": item_id}
```

## Path и Query параметры

```python
from fastapi import FastAPI, Path, Query

app = FastAPI()

@app.get("/items/{item_id}")
async def get_item(
    item_id: int = Path(
        ...,
        title="Item ID",
        ge=1,
        description="The ID of the item"
    ),
    q: str | None = Query(
        None,
        max_length=50,
        description="Search query"
    ),
    skip: int = Query(0, ge=0),
    limit: int = Query(10, ge=1, le=100),
):
    return {"item_id": item_id, "q": q, "skip": skip, "limit": limit}
```

## Request Body

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    name: str = Field(..., min_length=1, max_length=50)
    email: str
    age: int = Field(..., ge=0, le=150)
    is_active: bool = True

@app.post("/users/")
async def create_user(user: User):
    return user
```

## Response Model

```python
class UserOut(BaseModel):
    id: int
    name: str
    email: str

class UserIn(BaseModel):
    name: str
    email: str
    password: str

@app.post("/users/", response_model=UserOut)
async def create_user(user: UserIn):
    # password не попадёт в response
    return {"id": 1, "name": user.name, "email": user.email}
```

## Status Codes

```python
from fastapi import status

@app.post("/items/", status_code=status.HTTP_201_CREATED)
async def create_item(item: Item):
    return item

@app.delete("/items/{item_id}", status_code=204)
async def delete_item(item_id: int):
    return None
```

## Зависимости (зависимости)

```python
from fastapi import Depends, FastAPI

app = FastAPI()

def common_params(
    skip: int = 0,
    limit: int = 10
):
    return {"skip": skip, "limit": limit}

@app.get("/items/")
async def get_items(params: dict = Depends(common_params)):
    return {"params": params}
```

## Exceptions

```python
from fastapi import HTTPException

@app.get("/items/{item_id}")
async def get_item(item_id: int):
    if item_id < 0:
        raise HTTPException(
            status_code=400,
            detail="Invalid item ID"
        )
    if item_id > 100:
        raise HTTPException(
            status_code=404,
            detail="Item not found",
            headers={"X-Error": "Item error"}
        )
    return {"item_id": item_id}
```

## Best Practices

✅ **Используйте** `response_model` для фильтрации output
✅ **Используйте** `Field()` для валидации
✅ **Используйте** `status` модуль для status codes
✅ **Разделяйте** input/output модели

❌ **Не возвращайте** внутренние данные (password, secrets)
❌ **Не используйте** `Any` в response_model
❌ **Не логируйте** чувствительные данные

## Ссылки

- [Официальная документация FastAPI](https://fastapi.tiangolo.com/)
- [OpenAPI спецификация](https://www.openapis.org/)
- [Pydantic документация](https://docs.pydantic.dev/)

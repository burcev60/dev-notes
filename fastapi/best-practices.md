# FastAPI — Best Practices

## Описание

Рекомендации по структуре и паттернам FastAPI проектов.

## Структура проекта

```
app/
├── main.py
├── config.py
├── dependencies.py
├── api/
│   ├── v1/
│   │   ├── users.py
│   │   └── items.py
├── models/
│   ├── user.py
│   └── item.py
├── schemas/
│   ├── user.py
│   └── item.py
├── services/
│   ├── user_service.py
│   └── item_service.py
└── db/
    └── session.py
```

## APIRouter

```python
from fastapi import APIRouter

router = APIRouter(
    prefix="/users",
    tags=["users"],
    responses={404: {"description": "Not found"}},
)

@router.get("/")
async def get_users():
    return []

@router.post("/")
async def create_user():
    return {}
```

## Lifespan

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    await db.connect()
    yield
    # Shutdown
    await db.disconnect()

app = FastAPI(lifespan=lifespan)
```

## Best Practices

✅ **Используйте** APIRouter для модульности
✅ **Разделяйте** schemas/models/services
✅ **Используйте** lifespan для startup/shutdown

## Ссылки

- [FastAPI Best Practices](https://fastapi.tiangolo.com/tutorial/bigger-applications/)

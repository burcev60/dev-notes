# FastAPI — WebSockets

## Описание

FastAPI поддерживает WebSocket для двусторонней коммуникации.

## Базовое использование

```python
from fastapi import FastAPI, WebSocket

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Echo: {data}")
```

## WebSocket с авторизацией

```python
@app.websocket("/ws/{client_id}")
async def websocket_endpoint(
    websocket: WebSocket,
    client_id: int,
    token: str = Query(...)
):
    user = verify_token(token)
    await websocket.accept()
    await websocket.send_text(f"Hello, {user.name}!")
```

## Менеджер подключений

```python
class ConnectionManager:
    def __init__(self):
        self.active: list[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active.remove(websocket)

    async def broadcast(self, message: str):
        for ws in self.active:
            await ws.send_text(message)

manager = ConnectionManager()

@app.websocket("/ws/{client_id}")
async def websocket_endpoint(websocket: WebSocket, client_id: int):
    await manager.connect(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            await manager.broadcast(f"#{client_id}: {data}")
    except:
        manager.disconnect(websocket)
```

## Best Practices

✅ **Обрабатывайте** отключения
✅ **Используйте** менеджер для множества подключений

## Ссылки

- [FastAPI WebSockets](https://fastapi.tiangolo.com/advanced/websockets/)

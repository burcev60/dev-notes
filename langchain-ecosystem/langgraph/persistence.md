# LangGraph — Persistence и Store

## Описание

Сохранение и восстановление состояния графа через Checkpointer и persistent Store.

## Checkpointer

```python
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.checkpoint.base import BaseCheckpointSaver

# InMemorySaver — для разработки и тестов
checkpointer = InMemorySaver()

# Компиляция с checkpointer
graph = builder.compile(checkpointer=checkpointer)

# Вызов с thread_id
config = {"configurable": {"thread_id": "conversation_123"}}
result = graph.invoke({"messages": [HumanMessage("Hello")]}, config)

# Второй вызов — история сохраняется
result = graph.invoke({"messages": [HumanMessage("What did I say?")]}, config)
```

## Production Checkpointer

```python
# Для продакшена используйте:
# pip install langgraph-checkpoint-postgres

from langgraph.checkpoint.postgres import PostgresSaver
from langgraph.checkpoint.postgres.aio import AsyncPostgresSaver

# Синхронный
with PostgresSaver.from_dsn("postgresql://user:pass@localhost/db") as checkpointer:
    graph = builder.compile(checkpointer=checkpointer)

# Асинхронный
async with AsyncPostgresSaver.from_dsn("postgresql://...") as checkpointer:
    graph = builder.compile(checkpointer=checkpointer)
```

## Управление checkpoint

```python
# Получение истории checkpoint
checkpoints = list(graph.get_state_history(config))
for checkpoint in checkpoints:
    print(checkpoint.metadata)
    # {'source': 'input', 'step': 0, 'writes': {...}}
    # {'source': 'loop', 'step': 1, 'writes': {'assistant': ...}}

# Восстановление из конкретного checkpoint
graph.update_state(
    config,
    {"messages": [HumanMessage("Let me rephrase")]},
    as_node="assistant",  # от какого узла
)

# Fork — ветвление из checkpoint
fork_result = graph.invoke(
    {"messages": [HumanMessage("New direction")]},
    checkpoint_id=checkpoint.config["configurable"]["checkpoint_id"],
)
```

## Store — Persistent Key-Value хранилище

```python
from langgraph.store.base import BaseStore
from langgraph.store.memory import InMemoryStore

# In-memory store
store = InMemoryStore()

# Компиляция со store
graph = builder.compile(store=store)

# Доступ через runtime
from langgraph.runtime import Runtime, get_runtime

def save_user_prefs(state: State, runtime: Runtime) -> dict:
    store = runtime.store
    
    # Сохранение
    store.put(("users", state["user_id"]), "preferences", {
        "language": "ru",
        "theme": "dark",
    })
    
    # Получение
    prefs = store.get(("users", state["user_id"]), "preferences")
    
    # Поиск
    results = store.search(("users",), query="language")
    
    return {"preferences": prefs.value}

# Иерархические namespaces
store.put(("collection", "subcategory"), "item_id", {"data": "value"})
item = store.get(("collection", "subcategory"), "item_id")
results = store.search(("collection",), query="data")
```

## Store операции

```python
from langgraph.store.base import PutOp, GetOp, DeleteOp, SearchOp

# Batch операции
results = store.batch([
    PutOp(namespace=("users",), key="u1", value={"name": "Alice"}),
    GetOp(namespace=("users",), key="u1"),
    DeleteOp(namespace=("users",), key="u2"),
    SearchOp(namespace_prefix=("users",), query="Alice", filter=None, limit=10),
])

# TTL (время жизни)
store.put(
    ("sessions",), "session_123",
    {"data": "..."},
    ttl=3600,  # секунд
)
```

## Индексирование Store

```python
from langgraph.store.memory import InMemoryStore
from langchain_core.embeddings import Embeddings

# Store с семантическим поиском
store = InMemoryStore(
    index={
        "embed": embeddings,           # Функция векторизации
        "dims": 1536,                   # Размерность вектора
        "fields": ["content", "title"], # Поля для индексации
    }
)

# Семантический поиск
results = store.search(("docs",), query="Python programming")
```

## Checkpoint Metadata

```python
# Кастомная метаинформация
config = {
    "configurable": {
        "thread_id": "conv_123",
        "checkpoint_id": "...",
    },
    "metadata": {"user_id": "alice", "purpose": "testing"},
}

result = graph.invoke(input, config)

# Метаданные сохраняются в checkpoint
for cp in graph.get_state_history(config):
    print(cp.metadata.get("user_id"))  # "alice"
```

## Best Practices

✅ **Используйте** `InMemorySaver` только для разработки
✅ **Используйте** `PostgresSaver` для продакшена
✅ **Используйте** `store` для долгосрочных данных (preferences, profiles)
✅ **Используйте** `checkpoint` для краткосрочных данных (диалог)

❌ **Не храните** большие объёмы данных в checkpoint
❌ **Не используйте** in-memory checkpointer в продакшене
❌ **Не забывайте** указывать `thread_id` — иначе история не сохранится

## Ссылки

- [LangGraph Persistence](https://langchain-ai.github.io/langgraph/concepts/persistence/)
- [LangGraph Store](https://langchain-ai.github.io/langgraph/concepts/memory/)
- [LangGraph Checkpoint API](https://langchain-ai.github.io/langgraph/reference/checkpoints/)

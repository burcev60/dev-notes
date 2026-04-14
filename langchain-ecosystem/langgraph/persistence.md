# LangGraph — Persistence

## Описание

Сохранение и восстановление состояния графа через Checkpointer.

## Checkpointer

```python
from langgraph.checkpoint.memory import MemorySaver

checkpointer = MemorySaver()

app = workflow.compile(checkpointer=checkpointer)

# Сохранение
config = {"configurable": {"thread_id": "1"}}
result = app.invoke({"question": "Hello"}, config)

# Восстановление
result = app.invoke(None, config)
```

## SQLite Checkpointer

```python
from langgraph.checkpoint.sqlite import SqliteSaver

with SqliteSaver.from_conn_string("checkpoints.db") as checkpointer:
    app = workflow.compile(checkpointer=checkpointer)
```

## Best Practices

✅ **Используйте** persistence для долгосрочных диалогов
✅ **Используйте** thread_id для разделения сессий

## Ссылки

- [LangGraph Persistence](https://langchain-ai.github.io/langgraph/how-tos/persistence/)

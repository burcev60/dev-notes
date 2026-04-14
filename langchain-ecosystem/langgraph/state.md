# LangGraph — Управление состоянием

## Описание

Управление состоянием в LangGraph через TypedDict и аннотации.

## State с аннотациями

```python
from typing import Annotated
from langgraph.graph import StateGraph

class State(TypedDict):
    messages: Annotated[list, add_messages]
    context: dict

def process(state: State):
    state["messages"].append({"role": "assistant", "content": "Hello"})
    return state
```

## Best Practices

✅ **Используйте** Annotated для кастомных операций
✅ **Храните** минимум в состоянии

## Ссылки

- [LangGraph State](https://langchain-ai.github.io/langgraph/concepts/#state)

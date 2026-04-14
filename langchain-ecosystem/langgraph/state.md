# LangGraph — Управление состоянием

## Описание

Управление состоянием в LangGraph через TypedDict, аннотированные редюсеры и MessagesState.

## Annotated редюсеры

```python
from typing import Annotated
from typing_extensions import TypedDict
from langgraph.graph import add_messages

# Редюсер определяет как обновлять поле при возврате из узла
class State(TypedDict):
    # add_messages — добавляет сообщения, обновляет по ID, удаляет через RemoveMessage
    messages: Annotated[list, add_messages]
    
    # Кастомный редюсер
    counter: Annotated[int, lambda x, y: x + y]
    
    # Присваивание (замена значения)
    topic: str  # без Annotated — просто заменяется
```

## Встроенные редюсеры

```python
from langgraph.graph import add_messages
from operator import add

# add_messages — умное управление сообщениями
# - Добавляет новые в конец
# - Обновляет существующие по ID
# - Удаляет через RemoveMessage
# - Очищает через REMOVE_ALL_MESSAGES

# add (из operator) — суммирование
class State(TypedDict):
    total_score: Annotated[int, add]

# Кастомный редюсер
def merge_dicts(existing: dict, new: dict) -> dict:
    return {**existing, **new}

class State(TypedDict):
    metadata: Annotated[dict, merge_dicts]
```

## MessagesState — готовое состояние

```python
from langgraph.graph.message import MessagesState, MessageGraph

# MessagesState уже включает:
# messages: Annotated[list, add_messages]

# Наследование для добавления полей
class MyState(MessagesState):
    topic: str
    counter: int

def node(state: MyState) -> dict:
    # Возвращаем только обновления
    return {
        "messages": [AIMessage(content="Hello")],
        "topic": "greeting",
        "counter": state.get("counter", 0) + 1,
    }
```

## Работа с сообщениями

```python
from langchain_core.messages import (
    HumanMessage, AIMessage, RemoveMessage,
    trim_messages, filter_messages,
)

# Добавление сообщений
def assistant_node(state: MessagesState) -> dict:
    response = model.invoke(state["messages"])
    return {"messages": [response]}

# Удаление конкретного сообщения
def remove_node(state: MessagesState) -> dict:
    msg_id = state["messages"][0].id
    return {"messages": [RemoveMessage(id=msg_id)]}

# Усечение истории
def trim_node(state: MessagesState) -> dict:
    trimmed = trim_messages(
        state["messages"],
        max_tokens=4000,
        strategy="last",
        include_system=True,
    )
    return {"messages": trimmed}
```

## Input/Output Schema разделение

```python
class InputState(TypedDict):
    question: str  # Пользователь передаёт только вопрос

class OutputState(TypedDict):
    answer: str    # Пользователь получает только ответ

class FullState(TypedDict):
    question: str
    answer: str
    internal_notes: list[str]  # Внутреннее, не видно пользователю

builder = StateGraph(
    state_schema=FullState,
    input_schema=InputState,   # invoke({"question": "..."}) 
    output_schema=OutputState, # → {"answer": "..."}
)
```

## Node input schema

```python
# Узел с собственной схемой ввода
class SummarizeInput(TypedDict):
    messages: list
    topic: str

def summarize(state: SummarizeInput) -> dict:
    # Получает только указанные поля
    text = " ".join(m.content for m in state["messages"])
    summary = llm.invoke(f"Summarize: {text}")
    return {"answer": summary.content}

builder.add_node("summarize", summarize, input_schema=SummarizeInput)
```

## Best Practices

✅ **Используйте** `MessagesState` как базу для чат-ботов
✅ **Возвращайте** partial updates из узлов
✅ **Разделяйте** input_schema / output_schema для чистого API
✅ **Используйте** `Annotated` с редюсерами для сложной логики

❌ **Не возвращайте** полное состояние из узла (только изменения)
❌ **Не путайте** state_schema и input_schema (разные цели)
❌ **Не мутируйте** состояние напрямую — возвращайте dict обновлений

## Ссылки

- [LangGraph State Management](https://langchain-ai.github.io/langgraph/concepts/memory/)
- [LangGraph MessagesState](https://langchain-ai.github.io/langgraph/reference/graphs/#messagesstate)

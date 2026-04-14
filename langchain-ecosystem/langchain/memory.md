# LangChain — Memory и Chat History

## Описание

Управление историей сообщений в LangChain. Встроенные механизмы для хранения диалога и ограничения длины контекста.

## Базовая история сообщений

```python
from langchain_core.chat_history import (
    BaseChatMessageHistory,
    InMemoryChatMessageHistory,
)

# In-memory история
history = InMemoryChatMessageHistory()
history.add_messages([
    HumanMessage(content="Hello!"),
    AIMessage(content="Hi! How can I help?"),
])

print(history.messages)
history.clear()
```

## Сообщения

```python
from langchain_core.messages import (
    HumanMessage,     # Сообщение пользователя
    AIMessage,        # Ответ ассистента
    SystemMessage,    # Системная инструкция
    ToolMessage,      # Результат вызова инструмента
    ToolCall,         # Вызов инструмента
    RemoveMessage,    # Удаление сообщения (для add_messages редюсера)
    trim_messages,    # Усечение истории
    filter_messages,  # Фильтрация по типу
    merge_message_runs,  # Объединение последовательных сообщений
)

# Создание сообщений
human = HumanMessage(content="What is Python?")
ai = AIMessage(content="Python is a programming language.")
system = SystemMessage(content="You are a helpful assistant.")
tool = ToolMessage(content="42", tool_call_id="call_123")

# AI с tool calls
ai_with_tools = AIMessage(
    content="",
    tool_calls=[
        {"name": "search", "args": {"query": "Python"}, "id": "call_123"}
    ]
)
```

## trim_messages — Усечение истории

```python
from langchain_core.messages import trim_messages

# Ограничение по количеству токенов
trimmed = trim_messages(
    messages,
    max_tokens=4000,
    strategy="last",       # "last" | "first"
    token_counter=model,   # модель считает токены
    include_system=True,   # сохранять system message
    allow_partial=False,   # разрешать частичные сообщения
)

# Ограничение по количеству сообщений
trimmed = trim_messages(
    messages,
    max_tokens=10,  # последние 10 сообщений
    strategy="last",
)
```

## RunnableWithMessageHistory

```python
from langchain_core.runnables import RunnableWithMessageHistory
from langchain_core.chat_history import InMemoryChatMessageHistory

# Хранилище историй по session_id
store = {}

def get_session_history(session_id: str) -> BaseChatMessageHistory:
    if session_id not in store:
        store[session_id] = InMemoryChatMessageHistory()
    return store[session_id]

chain = prompt | model | StrOutputParser()

chain_with_history = RunnableWithMessageHistory(
    chain,
    get_session_history,
    input_messages_key="input",    # ключ input в prompt
    history_messages_key="history", # ключ history в prompt
)

result = chain_with_history.invoke(
    {"input": "What is my name?"},
    config={"configurable": {"session_id": "user_123"}},
)
```

## MessagesState (langgraph)

```python
from langgraph.graph.message import MessagesState, add_messages, REMOVE_ALL_MESSAGES
from typing import Annotated
from typing_extensions import TypedDict

# Предопределённое состояние с messages + add_messages редюсером
class MyState(MessagesState):
    extra_field: str

# add_messages редюсер автоматически:
# - Добавляет новые сообщения
# - Обновляет по ID
# - Удаляет через RemoveMessage
# - Очищает все через REMOVE_ALL_MESSAGES

# Custom состояние с messages
class CustomState(TypedDict):
    messages: Annotated[list, add_messages]
    context: dict
```

## Chat History в промптах

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant. Be concise."),
    MessagesPlaceholder("chat_history"),  # ← сюда подставляется история
    ("human", "{input}"),
])

chain = prompt | model | StrOutputParser()

result = chain.invoke({
    "chat_history": [
        HumanMessage(content="My name is Alice"),
        AIMessage(content="Hello Alice!"),
    ],
    "input": "What is my name?",
})
```

## Best Practices

✅ **Используйте** `trim_messages` для ограничения контекста
✅ **Используйте** `MessagesPlaceholder` для истории в промптах
✅ **Используйте** `RunnableWithMessageHistory` для сессий
✅ **Сохраняйте** system message при усечении

❌ **Не передавайте** неограниченную историю — токены кончатся
❌ **Не удаляйте** system message при trim

## Ссылки

- [LangChain Messages](https://python.langchain.com/docs/concepts/messages/)
- [LangChain Chat History](https://python.langchain.com/docs/how_to/#messages)
- [LangGraph MessagesState](https://langchain-ai.github.io/langgraph/concepts/memory/)
